# Dockerfile for MOM-CA

This is a Dockerfile with accompanying docker-compose.yml that enables the building of a Docker image for [MOM-CA](https://github.com/icaruseu/mom-ca). It is set up to use an existing [traefik](https://traefik.io/) container as reverse proxy.

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

| Name            | Default | Mandatory | Description                                                                   |
| --------------- | ------- | --------- | ----------------------------------------------------------------------------- |
| HOST            |         | yes       | The host name the reverse proxy listens to for connections to this container. |
| TRAEFIK_NETWORK | traefik | no        | The name of the external network used by traefik.                             |

## Example basic .env file

```
PASSWORD=my_password
MAX_MEMORY=4096
HOST=monasterium.net
TRAEFIK_NETWORK=traefik
```
