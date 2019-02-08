# Mini MPI

For this simple service, we use [PostgREST](http://postgrest.org/en/v5.2/).

The mini MPI consists of a single table with the following structure
	
	CREATE TABLE mpi.demographics (
	  id TEXT UNIQUE PRIMARY KEY,
	  nhs_number TEXT,
	  name TEXT,
	  surname TEXT,
	  pas_no TEXT,
	  phone TEXT,
	  address TEXT,
	  date_of_birth DATE,
	  department TEXT,
	  gender TEXT,
	  gp_address TEXT,
	  gp_name TEXT
	);

## Prerequisites

It is assumed that PostgreSQL is installed (see [https://tecadmin.net/install-postgresql-server-centos/](https://tecadmin.net/install-postgresql-server-centos/)) for a quick step by step instruction.

The following assumes the OS is CentOS 7.

## Creating the DB

Use the scripts in `mpi.sql`

# Deployment

Get the latest binary from PostgREST http://postgrest.org/en/v5.2/

## Installation

	[root@centos-s-1vcpu-1gb-lon1-01 src]# cd /usr/local/src
	[root@centos-s-1vcpu-1gb-lon1-01 src]# wget https://github.com/PostgREST/postgrest/releases/download/v5.2.0/postgrest-v5.2.0-centos7.tar.xz
	[root@centos-s-1vcpu-1gb-lon1-01 src]# tar xfJ postgrest-v5.2.0-centos7.tar.xz
	
	[root@centos-s-1vcpu-1gb-lon1-01 src]# mkdir /opt/postgrest
	[root@centos-s-1vcpu-1gb-lon1-01 src]# mkdir /opt/postgrest/bin
	[root@centos-s-1vcpu-1gb-lon1-01 src]# mv postgrest /opt/postgrest/bin
	[root@centos-s-1vcpu-1gb-lon1-01 src]# mkdir /etc/opt/postgrest

Configuration in `/etc/opt/postgrest/postgrest.conf`


	# postgrest.conf
	
	server-host = "*"
	
	# The standard connection URI format, documented at
	# https://www.postgresql.org/docs/current/static/libpq-connect.html#AEN45347
	db-uri       = "postgres://postgres:postgres@127.0.0.1:5432/mpi"
	
	# The name of which database schema to expose to REST clients
	db-schema    = "mpi"
	
	# The database role to use when no client authentication is provided.
	# Can (and probably should) differ from user in db-uri
	db-anon-role = "anonymous"
	
	# JWT secret
	jwt-secret = " *your secret key* "


**Don't forget to edit /var/lib/pgsql/10/data/pg_hba.conf and set local IPv4 to TRUST**

	host    all             all             127.0.0.1/32            trust

## Setting up and using JWT (from [https://jwt.io/](https://jwt.io/))

Header

	{
	  "alg": "HS256",
	  "typ": "JWT"
	}


Payload:

	{
	  "role": "mpi"
	}

Secret key:

	*your secret key*

The resulting token must be passed in the header parameter Authorization as

	Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJyb2xlIjoibXBpIn0.828rKR7CU5Ig_rLvGZ7fAqH89rq7iHg1...


## [Deamonizing](http://postgrest.org/en/v5.2/admin.html#daemonizing) 

The following configuration has been used

	[Unit]
	Description=REST API for any Postgres database
	After=postgresql.service
	
	[Service]
	ExecStart=/opt/postgrest/bin/postgrest /etc/opt/postgrest/postgrest.conf
	ExecReload=/bin/kill -SIGUSR1 $MAINPID
	
	[Install]
	WantedBy=multi-user.target

## PostgREST administration

Uses `systemctl cmd postgrest`

At this stage, logs are displayed with `systemctl status postgrest`

## Using PostgREST MPI

### Create a new entry

Use POST

	POST http://mpi-dev.ripple.foundation:3000/demographics
	[{"key":"Authorization","value":"Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJyb2xlIjoibXBpIn0.828rKR7CU5Ig_rLvGZ7fAqH89rq7iHg1...","description":""}]
	[{"key":"Content-Type","value":"application/json","description":""}]

Payload

	{

		"address": "6948 Et St., Halesowen, Worcestershire, VX27 5DV",
		"date_of_birth": "2012-03-27",
		"department": "Neighbourhood",
		"gender": "Male",
		"gp_address": "Hamilton Practice, 5544 Ante Street, Hamilton, Lanarkshire, N06 5LP",
		"gp_name": "Goff Carolyn D.",
		"id": "9999999000",
		"name": "Ivor",
		"surname": "Cox"
	}


### Retrieve all records

Use GET

	GET http://mpi-dev.ripple.foundation:3000/demographics
	[{"key":"Authorization","value":"Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJyb2xlIjoibXBpIn0.828rKR7CU5Ig_rLvGZ7fAqH89rq7iHg1...","description":""}]
	[{"key":"Content-Type","value":"application/json","description":""}]

==> result

	[
	    {
	        "id": "9999999000",
	        "nhs_number": null,
	        "name": "Ivor",
	        "pas_no": null,
	        "phone": null,
	        "address": "6948 Et St., Halesowen, Worcestershire, VX27 5DV",
	        "date_of_birth": "2012-03-27",
	        "department": "Neighbourhood",
	        "gender": "Male",
	        "gp_address": "Hamilton Practice, 5544 Ante Street, Hamilton, Lanarkshire, N06 5LP",
	        "gp_name": "Goff Carolyn D.",
	        "surname": "Cox"
	    }
	]

### Select a specific Id

	GET http://mpi-dev.ripple.foundation:3000/demographics?id=eq.9999999000

### select specific columns from a record

	GET http://mpi-dev.ripple.foundation:3000/demographics?id=eq.9999999000&select=nhs_number,name,pas_no

### Update example

Use PATCH

	PATCH http://mpi-dev.ripple.foundation:3000/demographics?id=eq.9999999000

Body
	
	{
		"name": "Ivory",
		"surname": "Coxy"
	}


Retrieve the record

	[
	    {
	        "id": "9999999000",
	        "nhs_number": null,
	        "name": "Ivory",
	        "pas_no": null,
	        "phone": null,
	        "address": "6948 Et St., Halesowen, Worcestershire, VX27 5DV",
	        "date_of_birth": "2012-03-27",
	        "department": "Neighbourhood",
	        "gender": "Male",
	        "gp_address": "Hamilton Practice, 5544 Ante Street, Hamilton, Lanarkshire, N06 5LP",
	        "gp_name": "Goff Carolyn D.",
	        "surname": "Coxy"
	    }
	]

### Delete example

	DELETE http://mpi-dev.ripple.foundation:3000/demographics?id=eq.9999999000

## Libraries

Several useful client API libraries are found at:

[http://postgrest.org/en/v5.2/index.html#clientside-libraries](http://postgrest.org/en/v5.2/index.html#clientside-libraries)


