FROM postgres:9.6
MAINTAINER Camptocamp <docker@camptocamp.com>

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update

RUN apt-get -y --no-install-suggests --no-install-recommends install \
     gcc \
     g++ \
     git \
     make \
     libffi-dev \
     curl \
     ca-certificates \
     rsync \
     libsnappy-dev \
     libssl-dev \
     python3 \
     python3-setuptools \
     python3-dev \
     libpq-dev

RUN curl -sSL https://bootstrap.pypa.io/get-pip.py | python3 \
  && pip3 install awscli boto python-keystoneclient python-swiftclient

# Install Gosu
ENV GOSU_VERSION 1.9
RUN set -x \
    && apt-get install -y --no-install-recommends ca-certificates wget && rm -rf /var/lib/apt/lists/* \
    && dpkgArch="$(dpkg --print-architecture | awk -F- '{ print $NF }')" \
    && wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch" \
    && wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch.asc" \
    && export GNUPGHOME="$(mktemp -d)" \
    && gpg --keyserver ha.pool.sks-keyservers.net --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 \
    && gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu \
    && rm -r "$GNUPGHOME" /usr/local/bin/gosu.asc \
    && chmod +x /usr/local/bin/gosu \
    && gosu nobody true

# Install confd
ENV CONFD_VERSION=0.12.0-alpha3  
RUN curl -L -o /usr/local/bin/confd https://github.com/kelseyhightower/confd/releases/download/v${CONFD_VERSION}/confd-${CONFD_VERSION}-linux-amd64 \
  && chmod +x /usr/local/bin/confd

# Install pghoard
ENV PGHOARD_VERSION=1.5.0
RUN pip3 install git+https://github.com/ohmu/pghoard.git@${PGHOARD_VERSION}

RUN pip3 install google-api-python-client 'oauth2client<4'

RUN mkdir /var/lib/pghoard
RUN chown postgres:postgres /var/lib/pghoard

RUN apt-get -y remove gcc python2.7 \
  && apt-get -y autoremove \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

ENV PGHOARD_BASEBACKUP_COUNT=1 \
  PGHOARD_BASEBACKUP_INTERVAL_HOURS=24 \
  PGHOARD_LOG_LEVEL=INFO \
  PGHOARD_STATSD_PORT=8125 \
  PGHOARD_STATSD_FORMAT=telegraf \
  PGHOARD_ACTIVE_BACKUP_MODE=pg_receivexlog

COPY conf.d /etc/confd/conf.d
COPY templates /etc/confd/templates
 
COPY /docker-entrypoint.sh /
WORKDIR /home/postgres

VOLUME [ "/home/postgres/pghoard" ]

ENTRYPOINT ["/docker-entrypoint.sh"]
