Docker Image for PostgreSQL 9.1.2 / PostGIS 1.5.8
=================================================

Why?
----
Because:

  - we want to reach a minimum level of proficiency with Docker, a wonderful
    technology new to us;

  - it's a developing and production environment setting we based a lot of
    projects on in the past, and we still use it in mantienance tasks.

What does this Docker image contains?
-------------------------------------
Compiled from source, this is what this image contains:

  - PostgreSQL 9.1.2;
  - PROJ 4.8.0, patched to support the NTv2 Spanish national grid for datum
    shiftings between ED50 and ETRS89;
  - GEOS 3.4.2;
  - PostGIS 1.5.8, also patched to support the spanish national grid.

Usage Pattern
-------------
Build the image directly from GitHub (this can take a while):

    docker build -t="geographica/postgresql-9.1.2-postgis-1.5.8"
	https://github.com/GeographicaGS/Docker-PostgreSQL-9.1.2-PostGIS-1.5.8.git

or pull it from Docker Hub:

    docker pull geographica/postgresql-9.1.2-postgis-1.5.8

Create a folder in the host to contain the data storage. We like to persist the
data storage in the host and not in the container:

    mkdir /whatever/postgresql-9.1.2-postgis-1.5.8-data

    chown postgres:malkab /whatever/postgresql-9.1.2-postgis-1.5.8-data

    chmod 700 /whatever/postgresql-9.1.2-postgis-1.5.8-data

Then create a temporary container to create the data storage. In the container,
/data will be always the data storage:

    docker run --rm -v /whatever/postgresql-9.1.2-postgis-1.5.8-data:/data/ -t
    -i geographica/postgresql-9.1.2-postgis-1.5.8 /bin/bash

    initdb --encoding=UTF-8 --locale=es_ES.UTF-8 --lc-collate=es_ES.UTF-8
    --lc-monetary=es_ES.UTF-8 --lc-numeric=es_ES.UTF-8 --lc-time=es_ES.UTF-8
	-D /data

(By the way, since I have a host-installed PostgreSQL of another version, I
can't tell the role the host __postgres__ user has in the permissions of the
data storage).

Modify access in the new data storage. In __postgresql.conf__, add universal
access from any IP:

    host    all             all             0.0.0.0/0               trust

Modify also the data storage to listen to all IP. In __postgresql.conf__,
modify:

    listen_addresses = '*'

Now we can exit the temporary container and create a new one that will use this
data storage:

    docker run -i -t --name="pgsql-9.1.2-postgis-1.5.8" -v
    /whatever/postgresql-9.1.2-postgis-1.5.8-data:/data/ -p 5455:5432
    geographica/postgresql-9.1.2-postgis-1.5.8

in this case, __-i__ and __-t__ are used so the container and the database can
be stoped with Ctl-C without __docker kill__ (I may be wrong, but I think this
is a cleaner way to exit the container, since the data storage will be properly
shut down).

This container will be permanent and we can start and attach to it in the usual
way:

    docker start pgsql-9.1.2-postgis-1.5.8

    docker attach pgsql-9.1.2-postgis-1.5.8

The server can be accessed the usual way, at port 5455:

    psql -h localhost -p 5455 -U postgres postgres
