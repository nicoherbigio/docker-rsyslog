FROM debian:bullseye

LABEL maintainer="Nico Herbig <nico@nicoherbig.io>"


# Add RSyslog user and group to ensure that the IDs (UID and GID)
# are consistently assigned regardless of which additional dependencies are added.

ENV RSYSLOG_USER_ID=999 RSYSLOG_USER=rsyslog
ENV RSYSLOG_GROUP_ID=999 RSYSLOG_GROUP=rsyslog

RUN set -ex \
    && groupadd -r -g ${RSYSLOG_GROUP_ID} ${RSYSLOG_GROUP} \
    && useradd -r -g ${RSYSLOG_GROUP} -u ${RSYSLOG_USER_ID} ${RSYSLOG_USER}


# Set time zone to "Europe/Berlin".

ENV TZ=Europe/Berlin

RUN set -ex \
    && apt-get update \
    && apt-get install -y --no-install-recommends tzdata \
    && rm -rf /var/lib/apt/lists/* \
    && ln -fns /usr/share/zoneinfo/${TZ} /etc/localtime \
    && echo ${TZ} > /etc/timezone


# Set locale to "en_US.UTF-8".

RUN set -ex \
    && if [ -f /etc/dpkg/dpkg.cfg.d/docker ]; then \
            # If this file exists, we are likely in "debian:xxx-slim" and locales are thus being excluded
            # so we need to remove that exclusion (since we need locales).
            grep -q '/usr/share/locale' /etc/dpkg/dpkg.cfg.d/docker; \
            sed -ri '/\/usr\/share\/locale/d' /etc/dpkg/dpkg.cfg.d/docker; \
            ! grep -q '/usr/share/locale' /etc/dpkg/dpkg.cfg.d/docker; \
       fi \
    && apt-get update \
    && apt-get install -y --no-install-recommends locales \
    && rm -rf /var/lib/apt/lists/* \
    && localedef -i de_DE -c -f UTF-8 -A /usr/share/locale/locale.alias de_DE.UTF-8 \
    && localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8

ENV LANG=en_US.UTF-8 LANGUAGE=en_US:en


# Install RSyslog, Logrotate and its dependencies.

RUN set -ex \
    && apt-get update \
    && apt-get install -y --no-install-recommends cron gzip logrotate rsyslog sudo tar mc \
    && rm -rf /var/lib/apt/lists/*


# Configure RSyslog, Logrotate and its dependencies.

COPY assets/etc/cron/logrotate /etc/cron.hourly/logrotate

COPY assets/etc/rsyslog/rsyslog.conf /etc/rsyslog.conf
COPY assets/etc/rsyslog/rsyslog_docker.conf /etc/rsyslog.d/rsyslog_docker.conf

RUN set -ex \
    && rm /etc/cron.daily/logrotate \
    && chown -R ${RSYSLOG_USER}:${RSYSLOG_GROUP} /etc/rsyslog.conf /etc/rsyslog.d/ /run /var/lib/logrotate/


# Configure Docker container startup, exposed ports and data volumes.

COPY assets/bin/docker-cmd /usr/local/bin/

RUN set -ex \
    && chmod +x /usr/local/bin/docker-cmd \
    && ln -s usr/local/bin/docker-cmd /

VOLUME /var/log/docker/

EXPOSE 601

CMD ["/docker-cmd"]
