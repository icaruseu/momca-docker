# Dockerfile for MOM-CA

This is a Dockerfile with accompanying docker-compose.yml that enables the building of a Docker image for [MOM-CA](https://github.com/icaruseu/mom-ca). It is set up to use an existing [traefik](https://traefik.io/) container as reverse proxy. If started with docker-compose file, the data is persisted in a named volume called _data_.

## Environment parameters

The following environment parameters can be seit either when building the Docker image as command line parameters or in a `.env` file used by docker-compose. There should be two folders parallel to the docker-compose file: `backup` and `restore`. These are mounted into the container and contain the backups MOM-CA makes as well as enables to provide backups for restoration purposes to the container.

### Build time

| Name             | Default                                | Mandatory | Description                                                 |
| ---------------- | -------------------------------------- | --------- | ----------------------------------------------------------- |
| BRANCH           | master                                 | no        | The git branch to use.                                      |
| REPOSITORY       | https://github.com/icaruseu/mom-ca.git | no        | The Git Repository to base the image on.                    |
| PASSWORD         |                                        | yes       | The admin password to set during build.                     |
| INIT_MEMORY      | 256                                    | no        | The initial memory available to the database.               |
| MAX_MEMORY       | 2048                                   | no        | The maximum memory available to the database.               |
| CACHE_SIZE       | 256                                    | no        | The eXist cache size.                                       |
| COLLECTION_CACHE | 256                                    | no        | The eXist collection cache size.                            |
| LUCENE_BUFFER    | 256                                    | no        | The eXist lucene buffer size.                               |
| BACKUP_TRIGGER   | 0 0 4 \* \* ?                          | no        | The definition for the backup trigger cronjob.              |
| REVISION         |                                        | no        | Enable the versioning system. Currently has no effect.      |
| HTTP_PORT        | 8080                                   | no        | The HTTP port the internal eXist Jetty server listens to.   |
| HTTPS_PORT       | 8443                                   | no        | The HTTPS port the internal eXist Jetty server listens to.  |
| SERVER_NAME      | localhost                              | no        | The name of the internal server.                            |
| USE_SSL          | false                                  | no        | Whether or not BetterFORM/eXist understands SSL connections |

### Run time

| Name            | Default | Mandatory           | Description                                                                   |
| --------------- | ------- | ------------------- | ----------------------------------------------------------------------------- |
| HOST            |         | yes for production  | The host name the reverse proxy listens to for connections to this container. |
| TRAEFIK_NETWORK | traefik | no                  | The name of the external network used by traefik.                             |
| DEV_DATA_PATH   |         | yes for development | The host path for the development data volume.                                |
| DEV_SRC_PATH    |         | yes for development | The host path for the development source volume.                              |

## Example basic .env file

The following has to be placed in a file named _.env_ next to the docker-compose.yml file.

```
PASSWORD=my_password
MAX_MEMORY=4096
HOST=monasterium.net
TRAEFIK_NETWORK=traefik
```

## Building image

Build the image using the following command:

```shell
sudo docker-compose build
```

### Using SSL

The basic parts of Monasterium are able to work with SSL connections without configuration apart from configuring Traefik in the correct way and setting the correct labels. Unfortunately, for the parts that use BetterFORM (creating new Fonds, importing Charters etc.), more configuration is necessary. The following steps need to be completed:

1) Provide valid SSL keys in a location on the parent file system. For example, this can be done using [traefik-certdumper](https://github.com/SvenDowideit/traefik-certdumper), a Docker container that converts the certificates used by traefik and usually stored in `acme.json` in different external formats. This enables working directly with the Let's Encrypt certificates that Traefik uses.
2) Mount the SSL keys as volumes. This can be done either by modifying `docker-compose.yml` or (preferably) by creating a `docker-compose.override.yml` that just contains the two additional volumes with an absolute path:

```
version: "3"
services:
  momca:
    volumes:
      - [Absolute path to public.pem]:/opt/momca/ssl/fullchain.pem:ro
      - [Absolute path to private.pem]:/opt/momca/ssl/privkey.pem:ro 
```

3) _Add `USE_SSL=true` in `.env`_
4) Build container with `sudo docker-compose build`

## Starting container

After the image is built, the container can be started with the following command:

```shell
sudo docker-compose up -d
```

**Please note that the container will only be available after 5 minutes via the configured host url due to the health check interval.**

## Restoring a backup

A backup can be restored by copying the file(s) to the _restore_ folder next to the docker-compose.yml file and running the following query (see [eXist documentation](https://exist-db.org/exist/apps/fundocs/view.html?uri=http://exist-db.org/xquery/syste)) in eXide:

```xquery
system:restore("/tmp/restore/[backup_name]", "[admin_password]", "[admin_password]")
```

## Running ant targets inside the docker container

Ant tasks defined in the [MOM-CA build.xml](https://github.com/icaruseu/mom-ca/blob/master/build.xml) can be called from outside with the following command.

```
sudo docker exec -it -w /opt/momca/mom.XRX momca ant [target]
```

## Stopping container

The container can be stopped with the following command:

```shell
sudo docker-compose down
```

## Log files

The log files will be exposed in the docker-compose repository under the `/logs` folder.

## Development with Docker

**Note: The development environment has to be cloned into and started from a different folder than a live environment on the same machine to avoid conflicts.**

This repository includes a docker-compose file suited for development. If started using the `docker-compose.dev.yml`, both the source and data directories of MOM-CA will be made available at a location configurable by environment variables (see above). The source code can then be directly modified in place and will be immediately visible inside the container. The code will be based on the git repository set in the environment variable used to build the image, so changes can be directly committed to the repository if so desired.

__Please note that the folders set in the environment variables need to be already existing before starting the container.__

Start the development container using the following command:

```shell
sudo docker-compose -f docker-compose.dev.yml up -d
```

MOM-CA will be available at the following url: _localhost:8080/mom/home_

To call an ant target inside the dev container use the following command:

```shell
sudo docker exec -it -w /opt/momca/mom.XRX momca-dev ant [target]
```

Stop the development server with the following command:

```shell
sudo docker-compose -f docker-compose.dev.yml down
```

_The data on the local file system will not be removed even if called with `down -v`._

### Development with Docker Desktop on Windows

Due to the different ways paths are handled on windows there are some things that need to be taken care for when running Docker with [Docker Desktop](https://docs.docker.com/docker-for-windows/install/), the preferred way to run Docker on Windows (with [hyper-v](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v) available):

1) Make sure that the drive(s) to be used in the _docker-compose.dev_ file is enabled in the Docker Desktop [shared drives settings](https://docs.docker.com/docker-for-windows/#shared-drives)
2) Add the path in the correct notation to the .env file, for instance: _/host_mnt/[drive-letter]/[path]_

Example _.env_ file for Docker Desktop on Windows
```
BRANCH=dev
PASSWORD=my-password
REPOSITORY=https://github.com/[user]/mom-ca.git

DEV_DATA_PATH=/host_mnt/d/temp/docker-mom.XRX-data
DEV_SRC_PATH=/host_mnt/d/projects/mom-ca
```
