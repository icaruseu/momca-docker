ARG BASE_CONTAINER=adoptopenjdk/openjdk12:latest
FROM $BASE_CONTAINER

LABEL maintainer="Daniel Jeller <djeller@nettek.at>"

USER root

ENV JAVA_TOOL_OPTIONS="-Dfile.encoding=UTF-8"

ARG BRANCH=master
ARG REPOSITORY="https://github.com/icaruseu/mom-ca.git"
ARG PASSWORD
ARG INIT_MEMORY=256
ARG MAX_MEMORY=2048
ARG CACHE_SIZE=256
ARG COLLECTION_CACHE=256
ARG LUCENE_BUFFER=256
ARG BACKUP_TRIGGER='0 0 4 * * ?'
ARG REVISION=''
ARG HTTP_PORT=8080
ARG HTTPS_PORT=8443
ARG SERVER_NAME='localhost'

RUN apt-get update && apt-get -yq dist-upgrade &&\
  apt-get install -yq --no-install-recommends \
  ant \
  git \
  sudo \
  unzip \
  lnav \
  nano \
  wget &&\
  rm -rf /var/lib/apt/lists/*

WORKDIR /momca

RUN git clone --single-branch --branch ${BRANCH} ${REPOSITORY} mom.XRX

WORKDIR /momca/mom.XRX

RUN rm build.properties.xml &&\
  echo '<?xml version="1.0" encoding="UTF-8"?>' >> build.properties.xml &&\
  echo '<!-- @author: Jochen Graf -->' >> build.properties.xml &&\
  echo '<XRX>' >> build.properties.xml &&\
  echo '<!-- ################################ XRX Project Settings ################################ -->' >> build.properties.xml &&\
  echo '<project-name>mom</project-name>' >> build.properties.xml &&\
  echo '<deploy>core/**,mom/**</deploy>' >> build.properties.xml &&\
  echo '<password>'"${PASSWORD}"'</password>' >> build.properties.xml &&\
  echo '<!-- ################################ Java Settings ################################ -->' >> build.properties.xml &&\
  echo '<java>' >> build.properties.xml &&\
  echo '<initmemory>'"${INIT_MEMORY}"'</initmemory>' >> build.properties.xml &&\
  echo '<maxmemory>'"${MAX_MEMORY}"'</maxmemory>' >> build.properties.xml &&\
  echo '</java>' >> build.properties.xml &&\
  echo '<!-- ################################ eXist Settings ################################ -->' >> build.properties.xml &&\
  echo '<exist>' >> build.properties.xml &&\
  echo '<cache-size>'"${CACHE_SIZE}"'</cache-size>' >> build.properties.xml &&\
  echo '<collection-cache>'"${COLLECTION_CACHE}"'</collection-cache>' >> build.properties.xml &&\
  echo '<lucene-buffer>'"${LUCENE_BUFFER}"'</lucene-buffer>' >> build.properties.xml &&\
  echo '<backup-cron-trigger>'"${BACKUP_TRIGGER}"'</backup-cron-trigger>' >> build.properties.xml &&\
  echo '<revision>'"${REVISION}"'</revision>' >> build.properties.xml &&\
  echo '</exist>' >> build.properties.xml &&\
  echo '<!-- ################################ Jetty Settings ################################ -->' >> build.properties.xml &&\
  echo '<jetty>' >> build.properties.xml &&\
  echo '<port>'"${HTTP_PORT}"'</port>' >> build.properties.xml &&\
  echo '<port-ssl>'"${HTTPS_PORT}"'</port-ssl>' >> build.properties.xml &&\
  echo '<servername>'"${SERVER_NAME}"'</servername>' >> build.properties.xml &&\
  echo '</jetty>' >> build.properties.xml &&\
  echo '</XRX>' >> build.properties.xml

RUN ant install &&\
  ant stop

WORKDIR /opt/momca

RUN mv /momca/* .

VOLUME /opt/momca/mom.XRX-data

RUN mkdir -p /tmp/restore &&\
  mkdir -p /opt/momca/mom.XRX-data/export

RUN export EXIST_HOME=/opt/momca/mom.XRX/localhost

RUN echo '#!/bin/bash' >> /usr/local/bin/docker-entrypoint.sh &&\
  echo 'set -e' >> /usr/local/bin/docker-entrypoint.sh &&\
  echo 'cd /opt/momca/mom.XRX' >> /usr/local/bin/docker-entrypoint.sh &&\
  echo 'sudo localhost/bin/startup.sh' >> /usr/local/bin/docker-entrypoint.sh &&\
  chmod +x /usr/local/bin/docker-entrypoint.sh

EXPOSE ${HTTP_PORT} ${HTTPS_PORT}

HEALTHCHECK --interval=5m --timeout=3s \
  CMD curl -f http://localhost:8080/ || exit 1

ENTRYPOINT ["docker-entrypoint.sh"]
