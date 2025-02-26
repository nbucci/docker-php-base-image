FROM php:7.2.0-fpm-alpine3.7

# Pass inexistent UUID (e.g.: 1001) to enhance the container security
ARG user=root

# Default php configurations values.
ENV PHP_MEMORY_LIMIT 64M
ENV PHP_TIMEZONE=Europe/Rome
ENV PHP_REALPATH_CACHE_SIZE 2M
ENV PHP_UPLOAD_MAX_FILE_SIZE 32M
ENV PHP_POST_MAX_SIZE 32M
ENV PHP_MAX_EXECUTION_TIME 30
ENV BLACKFIRE_HOST=${BLACKFIRE_HOST:-blackfire}

# FPM Configurations.
ENV PHP_FPM_MAX_CHILDREN 5
ENV PHP_FPM_START_SERVERS 2
ENV PHP_FPM_MIN_SPARE_SERVERS 1
ENV PHP_FPM_MAX_SPARE_SERVERS 3

ENV XDEBUG_VERSION 2.6.0alpha1
ENV MEMCACHE_VERSION 3.0.4
ENV PHPREDIS_VERSION 3.1.4
ENV PHP_EXTRA_DEPS libxml2-dev icu-dev libmemcached-dev cyrus-sasl-dev libpng libjpeg-turbo freetype-dev libpng-dev libjpeg-turbo-dev libwebp-dev
RUN apk update && \
    apk add ${PHP_EXTRA_DEPS} ${PHPIZE_DEPS} && \
    docker-php-ext-configure gd --with-gd --with-freetype-dir=/usr/include/ --with-png-dir=/usr/include/ --with-jpeg-dir=/usr/include/ --with-webp-dir=/usr/include && \
    apk add ${PHP_EXTRA_DEPS} ${PHPIZE_DEPS} gettext && \
    docker-php-ext-install bcmath gd intl mbstring pcntl pdo pdo_mysql soap zip && \
    pecl install xdebug-${XDEBUG_VERSION} && \
    pecl install redis-${PHPREDIS_VERSION} && \
    pecl install igbinary memcached-${MEMCACHE_VERSION} --disable-memcached-sasl && \
    apk del ${PHPIZE_DEPS} && \
    mkdir -p /tmp/blackfire && \
    curl -A "Docker" -L https://blackfire.io/api/v1/releases/client/linux_static/amd64 | tar zxp -C /tmp/blackfire && \
    mv /tmp/blackfire/blackfire /usr/bin/blackfire && \
    rm -Rf /tmp/blackfire && \
    rm -rf /var/cache/apk/*

# Install mailhog.
ENV MAILHOG_VERSION v0.1.9
RUN curl -fSL "https://github.com/mailhog/MailHog/releases/download/${MAILHOG_VERSION}/MailHog_linux_amd64" -o /usr/local/bin/mailhog \
    && chmod +x /usr/local/bin/mailhog

# Configure entrypoint.
COPY docker-entrypoint.sh /docker-entrypoint.sh
RUN chmod +x /docker-entrypoint.sh

# Configure PHP.
COPY conf/*.ini /usr/local/etc/php/conf.d/
COPY conf.disabled /usr/local/etc/php/conf.disabled
COPY fpm-conf-templates/ /templates/

# Make folders writable for the root group
RUN chmod 775 /usr/local/etc/php && \
    chmod 775 /usr/local/etc/php/conf.d && \
    chmod 775 /usr/local/etc/php-fpm.d && \
    chmod 775 /templates

# Go to target user
USER $user

ENTRYPOINT [ "/docker-entrypoint.sh" ]
CMD ["php-fpm"]
