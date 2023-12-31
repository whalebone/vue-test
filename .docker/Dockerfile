FROM dockette/debian:buster

# PHP
ENV PHP_MODS_DIR=/etc/php/8.1/mods-available
ENV PHP_CLI_DIR=/etc/php/8.1/cli/
ENV PHP_CLI_CONF_DIR=${PHP_CLI_DIR}/conf.d
ENV PHP_CGI_DIR=/etc/php/8.1/cgi/
ENV PHP_CGI_CONF_DIR=${PHP_CGI_DIR}/conf.d
ENV PHP_FPM_DIR=/etc/php/8.1/fpm/
ENV PHP_FPM_CONF_DIR=${PHP_FPM_DIR}/conf.d
ENV PHP_FPM_POOL_DIR=${PHP_FPM_DIR}/pool.d
ENV TZ=Europe/Prague

# INSTALLATION
RUN apt update && apt dist-upgrade -y && \
    # DEPENDENCIES #############################################################
    apt install -y wget curl apt-transport-https ca-certificates gnupg2 && \
    # PHP DEB.SURY.CZ ##########################################################
    wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg && \
    echo "deb https://packages.sury.org/php/ buster main" > /etc/apt/sources.list.d/php.list && \
    wget -O- http://nginx.org/keys/nginx_signing.key | apt-key add - && \
    echo "deb http://nginx.org/packages/debian/ buster nginx" > /etc/apt/sources.list.d/nginx.list && \
    echo "deb-src http://nginx.org/packages/debian/ buster nginx" >> /etc/apt/sources.list.d/nginx.list && \
    apt update && \
    apt install -y --no-install-recommends \
        nginx \
        supervisor \
        php8.1-apc \
        php8.1-apcu \
        php8.1-bz2 \
        php8.1-bcmath \
        php8.1-calendar \
        php8.1-cgi \
        php8.1-cli \
        php8.1-ctype \
        php8.1-curl \
        php8.1-fpm \
        php8.1-gettext \
        php8.1-gd \
        php8.1-intl \
        php8.1-imap \
        php8.1-ldap \
        php8.1-mbstring \
        php8.1-memcached \
        php8.1-mongo \
        php8.1-mysql \
        php8.1-pdo \
        php8.1-pgsql \
        php8.1-redis \
        php8.1-soap \
        php8.1-sqlite3 \
        php8.1-ssh2 \
        php8.1-tidy \
        php8.1-zip \
        php8.1-xmlrpc \
        php8.1-xsl && \
    # COMPOSER #################################################################
    curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer --2 && \
    # NGINX ####################################################################
    ln -sf /dev/stdout /var/log/nginx/access.log && \
    ln -sf /dev/stderr /var/log/nginx/error.log && \
    # CLEAN UP #################################################################
    rm /etc/nginx/conf.d/default.conf && \
    apt-get clean -y && \
    apt-get autoclean -y && \
    apt-get remove -y wget curl && \
    apt-get autoremove -y && \
    rm -rf /var/lib/apt/lists/* /var/lib/log/* /tmp/* /var/tmp/*

# PHP
ADD .docker/php/php-fpm.conf /etc/php/8.1/

# NGINX
ADD .docker/nginx/nginx.conf /etc/nginx/
ADD .docker/nginx/mime.types /etc/nginx/
ADD .docker/nginx/sites.d /etc/nginx/sites.d


# SUPERVISOR
ADD .docker/supervisor/supervisord.conf /etc/supervisor/
ADD .docker/supervisor/services /etc/supervisor/conf.d/

ADD ../app/ /srv/www/app/
ADD ../www /srv/www/www/
ADD ../config /srv/www/config/
ADD ../composer.json /srv/www/
ADD ../composer.lock /srv/www/

# APPLICATION
WORKDIR /srv/www

RUN mkdir -p ./temp && mkdir -p ./log && composer install

# PORTS
EXPOSE 80

ENV PHP_EXTENSION_MYSQLI=1
ENV STARTUP_COMMAND_1="composer install"

CMD ["/bin/bash", "-c", "chmod o+w /srv/www/log/ /srv/www/temp/ && supervisord --nodaemon --configuration /etc/supervisor/supervisord.conf"]
