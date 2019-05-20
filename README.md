# Dockerfile for MOM-CA

This is a Dockerfile with accompanying docker-compose.yml that enables the building of a Docker image for [MOM-CA](https://github.com/icaruseu/mom-ca). It is set up to use an existing [traefik](https://traefik.io/) container as reverse proxy. If started with docker-compose file, the data is persisted in a named volume called _data_.

## Install Docker, docker-compose ant traefik

* [Docker](https://docs.docker.com/install/)
* [docker-compose](https://docs.docker.com/compose/install)
* [traefik](https://docs.traefik.io/user-guide/docker-and-lets-encrypt/)

## Environment parameters

The following environment parameters can be seit either when building the Docker image as command line parameters or in a `.env` file used by docker-compose. There should be two folders parallel to the docker-compose file: `backup` and `restore`. These are mounted into the container and contain the backups MOM-CA makes as well as enables to provide backups for restoration purposes to the container.

### Build time

| Name             | Default                                | Mandatory | Description                                                |
| ---------------- | -------------------------------------- | --------- | ---------------------------------------------------------- |
| BRANCH           | master                                 | no        | The git branch to use.                                     |
| REPOSITORY       | https://github.com/icaruseu/mom-ca.git | no        | The Git Repository to base the image on.                   |
| PASSWORD         |                                        | yes       | The admin password to set during build.                    |
| INIT_MEMORY      | 256                                    | no        | The initial memory available to the database.              |
| MAX_MEMORY       | 2048                                   | no        | The maximum memory available to the database.              |
| CACHE_SIZE       | 256                                    | no        | The eXist cache size.                                      |
| COLLECTION_CACHE | 256                                    | no        | The eXist collection cache size.                           |
| LUCENE_BUFFER    | 256                                    | no        | The eXist lucene buffer size.                              |
| BACKUP_TRIGGER   | 0 0 4 \* \* ?                          | no        | The definition for the backup trigger cronjob.             |
| REVISION         |                                        | no        | Enable the versioning system. Currently has no effect.     |
| HTTP_PORT        | 8080                                   | no        | The HTTP port the internal eXist Jetty server listens to.  |
| HTTPS_PORT       | 8443                                   | no        | The HTTPS port the internal eXist Jetty server listens to. |
| SERVER_NAME      | localhost                              | no        | The name of the internal server.                           |

### Run time

| Name            | Default | Mandatory           | Description                                                                   |
| --------------- | ------- | ------------------- | ----------------------------------------------------------------------------- |
| HOST            |         | yes for production  | The host name the reverse proxy listens to for connections to this container. |
| TRAEFIK_NETWORK | web     | no                  | The name of the external network used by traefik.                             |
| DEV_DATA_PATH   |         | yes for development | The host path for the development data volume.                                |
| DEV_SRC_PATH    |         | yes for development | The host path for the development source volume.                              |

## Example basic .env file

```
PASSWORD=my_password
MAX_MEMORY=4096
HOST=monasterium.net
TRAEFIK_NETWORK=web
```

## Building image

Build the image using the following command:

```shell
sudo docker-compose build
```

## Starting container

After the image is built, the container can be started with the following command:

```shell
sudo docker-compose up -d
```

**Please note that the container will only be available after 5 minutes via the configured host url due to the health check interval.**

## Restoring a backup

A backup can be restored by copying the file(s) to the _restore_ folder next to the docker-compose.yml file and running the following query in eXide:

```xquery
system:restore("/tmp/restore/[backup_name]", "[admin_password]", "admin_password")
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

## Development with Docker

**Note: The development environment has to be cloned into and started from a different folder than a live environment on the same machine to avoid conflicts.**

This repository includes a docker-compose file suited for development. If started using the `docker-compose.dev.yml`, both the source and data directories of MOM-CA will be made available at a location configurable by environment variables (see above). The source code can then be directly modified in place and will be immediately visible inside the container. The code will be based on the git repository set in the environment variable used to build the image, so changes can be directly committed to the repository if so desired.

__Please note that the folders set in the environment variables need to be already existing before starting the container.__

Start the development container using the following command:

```shell
sudo docker-compose -f docker-compose.dev.yml up -d
```

After the modification of the source files and building the code with `ant install` (see below) or similar, the data can be accessed directly in _./dev/mom.XRX-data_.

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
