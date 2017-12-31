FROM meg4r0m/docker-alpine-php:7.2

LABEL nginx.version=1.12 \
      php.version=7.2 \
      application.type=PHP-FPM
LABEL maintainer="Florian Durano <psykoterro@gmail.com>" \
    org.label-schema.schema-version="1.0" \
    org.label-schema.vendor="meg4r0m" \
    org.label-schema.name="nginx-php-alpine" \
    org.label-schema.description="small nginx-php image based on alpine" \
    org.label-schema.vcs-url="https://github.com/psykoterro/docker-php-alpine"

ARG DOCKER_TIMEZONE=Europe/Paris
ARG DOCKER_LOCALE=fr_FR.UTF-8

# Configure timezone
# -----------------------------------------------------------------------------
RUN apk add --update tzdata && \
	cp /usr/share/zoneinfo/$DOCKER_TIMEZONE /etc/localtime && \
    echo $DOCKER_TIMEZONE > /etc/timezone

# Base packages
# Java (used for eg. eZTika indexing)
# poppler-utils used for pdftotext binary (note that eztika is a much better option!)
# -----------------------------------------------------------------------------
RUN apk add --no-cache \
    pwgen \
    sudo \
    nano \
    unzip \
    imagemagick \
    jpegoptim \
    poppler-utils \
    openjdk7-jre;

# Install language pack
RUN apk --no-cache add ca-certificates wget && \
    wget -q -O /etc/apk/keys/sgerrand.rsa.pub https://raw.githubusercontent.com/sgerrand/alpine-pkg-glibc/master/sgerrand.rsa.pub && \
    wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.25-r0/glibc-2.25-r0.apk && \
    wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.25-r0/glibc-bin-2.25-r0.apk && \
    wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.25-r0/glibc-i18n-2.25-r0.apk && \
    apk add glibc-bin-2.25-r0.apk glibc-i18n-2.25-r0.apk glibc-2.25-r0.apk

# Iterate through all locale and install it
# Note that locale -a is not available in alpine linux, use `/usr/glibc-compat/bin/locale -a` instead
COPY ./locale.md /locale.md
RUN cat locale.md | xargs -i /usr/glibc-compat/bin/localedef -i {} -f UTF-8 {}.UTF-8

# locale for date, time & currency php functions
# -----------------------------------------------------------------------------
ENV LC_ALL $DOCKER_LOCALE
ENV LANG $DOCKER_LOCALE
ENV LANGUAGE $DOCKER_LOCALE

# Install Nginx & PHP
# -----------------------------------------------------------------------------
RUN echo "@edge-main http://dl-cdn.alpinelinux.org/alpine/edge/main" >> /etc/apk/repositories \
    && echo "@edge-community http://dl-cdn.alpinelinux.org/alpine/edge/community" >> /etc/apk/repositories \
    && echo "@cast http://php.codecasts.rocks/v3.7/php-7.2" >> /etc/apk/repositories

RUN apk add nginx \
    php7-fpm@cast \
    bash \
    curl

# Local user
# => to try : run container with --user option to set root user id with host user id
# -----------------------------------------------------------------------------
# nb: the 1013 used here for user id and group id is later on replaced by the code in bootstrap.sh...
RUN addgroup -g 1013 site && \
    adduser -S -u 1013 -G site site \
    -h /home/site -s /bin/bash site && \
    adduser site site && \
    adduser site www-data && \
    mkdir -p /home/site/.ssh; \
    cp /etc/skel/.* /home/site/; \
    chown -R site:site /home/site;

# Nginx config
# -----------------------------------------------------------------------------
RUN sed -i -e"s/worker_processes  1/worker_processes 4/" /etc/nginx/nginx.conf && \
    sed -i -e"s/keepalive_timeout\s*65/keepalive_timeout 2/" /etc/nginx/nginx.conf && \
    sed -i -e"s/keepalive_timeout 2/keepalive_timeout 2/" /etc/nginx/nginx.conf && \
    sed -i -e"s/client_max_body_size 1m/client_max_body_size 100m/" /etc/nginx/nginx.conf && \
    sed -i -e's|user www-data;|user site;|g' /etc/nginx/nginx.conf && \
    chmod 755 -R /var/log/nginx

# PHP config
# -----------------------------------------------------------------------------
# We set a unique service name for php-fpm to be able to use the same bootstrap script
RUN sed -rie 's|user = nobody|user = site|g' /etc/php7/php-fpm.d/www.conf && \
    sed -rie 's|;listen.owner = nobody|listen.owner = site|g' /etc/php7/php-fpm.d/www.conf && \
    sed -rie 's|;listen.group = nobody|listen.group = site|g' /etc/php7/php-fpm.d/www.conf && \
    sed -rie 's|;listen.mode = 0660|listen.mode = 0660|g' /etc/php7/php-fpm.d/www.conf && \
    sed -rie 's|;pid = run\/php-fpm7.pid|pid = \/var\/run\/php\/php-fpm.sock|g' /etc/php7/php-fpm.conf && \
    mv /etc/init.d/php-fpm7 /etc/init.d/php-fpm

# Disable xdebug
#RUN rm /etc/php7/conf.d/00_xdebug.ini

# Custom PHP-FPM error log folder
RUN mkdir /var/log/php/ && \
    chmod 755 -R /var/log/php/ && \
    sed -rie 's|error_log = /var/log/php7.0-fpm.log|error_log = /var/log/php/php7.0-fpm.log|g' /etc/php7/php-fpm.conf

# Vhost config
# -----------------------------------------------------------------------------
# default & controlpanel
COPY conf/vhosts/010-default /etc/nginx/sites-enabled/010-default
COPY conf/vhosts/010-controlpanel.conf /etc/nginx/conf.d/010-controlpanel.conf

COPY build_files/controlpanel.tar.gz /tmp/controlpanel.tar.gz
RUN tar -zxf /tmp/controlpanel.tar.gz -C /home/site/ && \
    rm /tmp/controlpanel.tar.gz

# Run composer install for phpmemadmin vendors
RUN cd /home/site/controlpanel/phpmemadmin && \
    curl -sS https://getcomposer.org/installer | php -- && \
    php composer.phar install --no-dev --ignore-platform-reqs
    
# dynamic vhosts
# a2ensite is run in bootstrap.sh for these as they could also be mounted as volume at run time
COPY conf/vhosts/001-unique-vhost /etc/nginx/sites-enabled/001-unique-vhost
COPY conf/vhosts/002-dynamic-vhost-ez5 /etc/nginx/sites-enabled/002-dynamic-vhost-ez5
COPY conf/vhosts/003-dynamic-vhost-ez4 /etc/nginx/sites-enabled/003-dynamic-vhost-ez4
COPY conf/vhosts/ez5-common /etc/nginx/ez5-common
COPY conf/vhosts/environment_common /etc/nginx/environment_common
COPY conf/vhosts/fastcgi_common /etc/nginx/fastcgi_common
# -----------------------------------------------------------------------------

RUN mkdir -p /run/nginx
RUN mkdir -p /run/php

# Clear archives in cache folder
RUN rm -rf /var/cache/apk/*

COPY conf/bootstrap.sh /root/bootstrap.sh
RUN chmod 755 /root/bootstrap.sh

EXPOSE 443
EXPOSE 80
EXPOSE 82

WORKDIR /var/www/

CMD ["/root/bootstrap.sh"]