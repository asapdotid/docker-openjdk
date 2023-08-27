FROM azul/zulu-openjdk-alpine:20-jre-headless

# Metadata
LABEL maintainer "Asapdotid <asapdotid@gmail.com>"

ENV TERM="xterm" \
    LANG="C.UTF-8" \
    LC_ALL="C.UTF-8"

ENV APPLICATION_USER=application \
    APPLICATION_GROUP=application \
    APPLICATION_PATH=/app \
    APPLICATION_UID=1000 \
    APPLICATION_GID=1000

RUN addgroup -g ${APPLICATION_GID} ${APPLICATION_USER} \
    && adduser -u ${APPLICATION_UID} -G ${APPLICATION_GROUP} -s /bin/sh -D ${APPLICATION_USER} \
    && apk --update --no-cache add supervisor nodejs npm >/dev/null

# Set timezone
ENV TZ ${TIMEZONE:-Etc/UTC}
RUN cp /usr/share/zoneinfo/$TZ /etc/localtime

# Setup supervisor
RUN mkdir -p /etc/supervisor/conf.d
ADD src/openjdk/20/conf/supervisord.conf /etc/supervisord.conf

# Install pm2
RUN npm install pm2 -g

RUN npm cache clean --force --loglevel=error \
    && rm -rf /var/cache/apk/* \
    && rm -rf /tmp/*

WORKDIR ${APPLICATION_PATH}

USER ${APPLICATION_USER}

# default command
CMD [ "java", "-version" ]
