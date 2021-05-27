# Dockerfile for ELK stack
# Elasticsearch, Logstash, Kibana 7.11.2

# Build with:
# docker build -t <repo-user>/elk .

# Run with:
# docker run -p 5601:5601 -p 9200:9200 -p 5044:5044 -it --name elk <repo-user>/elk

# replace with master-arm64 for ARM64
ARG IMAGE=18.04-1.0.0

FROM phusion/baseimage:${IMAGE}
MAINTAINER Sebastien Pujadas http://pujadas.net
ENV \
    REFRESHED_AT=2020-06-20


###############################################################################
#                                INSTALLATION
###############################################################################

### install prerequisites (cURL, gosu, tzdata, JDK for Logstash)

RUN set -x \
    && apt update -qq \
    && apt install -qqy --no-install-recommends ca-certificates curl gosu tzdata openjdk-11-jdk-headless \
    && apt clean \
    && rm -rf /var/lib/apt/lists/* \
    && gosu nobody true \
    && set +x


### set current package version

ARG ELK_VERSION=7.11.2

# base version (i.e. remove OSS prefix) for Elasticsearch and Kibana (no OSS version since 7.11.0)
ARG ELK_BASE_VERSION=7.11.2

# replace with aarch64 for ARM64 systems
ARG ARCH=x86_64


### install Elasticsearch

# predefine env vars, as you can't define an env var that references another one in the same block
ENV \
    ES_VERSION=${ELK_BASE_VERSION} \
    ES_HOME=/opt/elasticsearch

ENV \
    ES_PACKAGE=elasticsearch-${ES_VERSION}-linux-${ARCH}.tar.gz \
    ES_GID=991 \
    ES_UID=991 \
    ES_PATH_CONF=/etc/elasticsearch \
    ES_PATH_BACKUP=/var/backups

RUN DEBIAN_FRONTEND=noninteractive \
    && mkdir ${ES_HOME} \
    && curl -O https://artifacts.elastic.co/downloads/elasticsearch/${ES_PACKAGE} \
    && tar xzf ${ES_PACKAGE} -C ${ES_HOME} --strip-components=1 \
    && rm -f ${ES_PACKAGE} \
    && groupadd -r elasticsearch -g ${ES_GID} \
    && useradd -r -s /usr/sbin/nologin -M -d ${ES_HOME} -c "Elasticsearch service user" -u ${ES_UID} -g elasticsearch elasticsearch \
    && mkdir -p /var/log/elasticsearch ${ES_PATH_CONF} ${ES_PATH_CONF}/scripts /var/lib/elasticsearch ${ES_PATH_BACKUP} \
    && chown -R elasticsearch:elasticsearch ${ES_HOME} /var/log/elasticsearch /var/lib/elasticsearch ${ES_PATH_CONF} ${ES_PATH_BACKUP}

###############################################################################
#                              START-UP SCRIPTS
###############################################################################

### Elasticsearch

ADD ./elasticsearch-init /etc/init.d/elasticsearch
RUN sed -i -e 's#^ES_HOME=$#ES_HOME='$ES_HOME'#' /etc/init.d/elasticsearch \
    && chmod +x /etc/init.d/elasticsearch


###############################################################################
#                               CONFIGURATION
###############################################################################

### configure Elasticsearch

ADD ./elasticsearch.yml ${ES_PATH_CONF}/elasticsearch.yml
ADD ./elasticsearch-default /etc/default/elasticsearch
RUN cp ${ES_HOME}/config/log4j2.properties ${ES_HOME}/config/jvm.options \
    ${ES_PATH_CONF} \
    && chown -R elasticsearch:elasticsearch ${ES_PATH_CONF} \
    && chmod -R +r ${ES_PATH_CONF}

### configure logrotate

ADD ./elasticsearch-logrotate /etc/logrotate.d/elasticsearch
RUN chmod 644 /etc/logrotate.d/elasticsearch

###############################################################################
#                                   START
###############################################################################

ADD ./start.sh /usr/local/bin/start.sh
RUN chmod +x /usr/local/bin/start.sh

EXPOSE 5601 9200 9300 9600 5044
VOLUME /var/lib/elasticsearch

CMD [ "/usr/local/bin/start.sh" ]
