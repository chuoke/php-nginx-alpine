FROM alpine:3.21

ARG ALPINE_VERSION=3.21

LABEL Maintainer="chuoke"

# mirrors.aliyun.com
# mirrors.tuna.tsinghua.edu.cn
ARG APPEND_ALPINE_APK_REPO=mirrors.aliyun.com
ENV APPEND_ALPINE_APK_REPO=${APPEND_ALPINE_APK_REPO}

# RUN if [ ${ALPINE_APK_REPO} ]; then \
#         sed -i "s/dl-cdn.alpinelinux.org/${ALPINE_APK_REPO}/g" /etc/apk/repositories; \
#     fi

RUN if [ ${APPEND_ALPINE_APK_REPO} ]; then \
        echo "https://${APPEND_ALPINE_APK_REPO}/alpine/v${ALPINE_VERSION}/main" >> /etc/apk/repositories; \
        echo "https://${APPEND_ALPINE_APK_REPO}/alpine/v${ALPINE_VERSION}/community" >> /etc/apk/repositories; \
    fi

# RUN apk update && \
#     apk upgrade --available && \
RUN apk update && apk add --no-cache \
    openssl bash curl ca-certificates shadow \
    nodejs

## Install about PHP

## Many extensions are included in the base package,
## but in order to ensure that the required extensions must exist,
## all of them are mandatory installation

RUN apk add --no-cache \
    php84 \
    php84-common \
    php84-fpm \
    # Laravel Requirements
    php84-bcmath \
    php84-bz2 \
    php84-ctype \
    php84-fileinfo \
    php84-json \
    php84-mbstring \
    php84-openssl \
    php84-pdo \
    php84-tokenizer \
    php84-xml \
    # Base Requirements
    php84-curl \
    php84-dom \
    php84-gd \
    php84-opcache \
    php84-xmlwriter \
    php84-simplexml \
    php84-posix \
    php84-opcache \
    # Other Requirements
    php84-zlib \
    php84-zip \
    php84-sodium \
    php84-phar \
    php84-exif \
    php84-iconv \
    php84-intl \
    php84-session \
    php84-xmlreader \
    php84-ftp \
    php84-sockets \
    php84-calendar \
    php84-mysqli \
    php84-pdo_mysql \
    php84-pdo_sqlite \
    php84-sqlite3 \
    php84-pcntl \
    php84-gettext \
    php84-imap \
    # pecl extensions
    php84-pecl-apcu \
    php84-pecl-redis \
    php84-pecl-mongodb \
    php84-pecl-imagick \
    php84-pecl-xdebug \
    php84-pecl-swoole

# Configure PHP-FPM
COPY php-fpm.conf /etc/php84/php-fpm.d/www.conf
COPY php.ini /etc/php84/conf.d/custom.ini

RUN rm -f /usr/bin/php && ln -s /usr/bin/php84 /usr/bin/php && \
    ln -s /usr/sbin/php-fpm84 /usr/sbin/php-fpm

# ----------------------------------------------------------------
# Install composer

# Install composer from apk (size bigger than following 2 ways)
# RUN apk add composer

# Install composer manually
# ENV COMPOSER_HOME=/composer
# ENV PATH=./vendor/bin:/composer/vendor/bin:$PATH
# ENV COMPOSER_ALLOW_SUPERUSER=1
# RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" && \
#     php composer-setup.php --install-dir=/usr/local/bin --filename=composer && \
#     php -r "unlink('composer-setup.php');"
# Install composer from composer docker image
COPY --from=composer:latest /usr/bin/composer /usr/local/bin/composer

# Config Composer packagist repo
# ARG COMPOSER_REPO_PACKAGIST=https://mirrors.aliyun.com/composer/
# ARG COMPOSER_REPO_PACKAGIST=https://mirrors.tencent.com/composer/
# ENV COMPOSER_REPO_PACKAGIST=${COMPOSER_REPO_PACKAGIST}
# RUN if [ ${COMPOSER_REPO_PACKAGIST} ]; then \
#     composer config -g repo.packagist composer ${COMPOSER_REPO_PACKAGIST} \
# ;fi


# install roadrunner
# COPY --from=ghcr.io/roadrunner-server/roadrunner:latest /usr/bin/rr /usr/local/bin/rr


# # install frankenphp
# RUN apk add jq
# RUN wget -O/usr/local/bin/frankenphp $(wget -O- https://api.github.com/repos/dunglas/frankenphp/releases/latest | jq '.assets[] | select(.name=="frankenphp-linux-x86_64") | .browser_download_url' -r) && chmod +x /usr/local/bin/frankenphp


# Install about nginx
RUN apk --no-cache add nginx && \
    if [ -f '/etc/nginx/conf.d/default.conf' ]; then \
        rm /etc/nginx/conf.d/default.conf \
    ;fi

# Configure nginx
COPY nginx.conf /etc/nginx/nginx.conf
COPY nginx-server-temps /etc/nginx/server-temps

# Install supervisor
RUN apk --no-cache add supervisor
# RUN apk add py3-setuptools py3-pip && pip install supervisor
# Configure supervisord
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY supervisord-site.conf /etc/supervisor/conf.d/supervisord.d/site.conf

# Clean
RUN rm -rf /var/cache/apk/*

# ensure www-data user exists
RUN set -x ; \
  addgroup -g 82 -S www-data ; \
  adduser -u 82 -D -S -G www-data www-data && exit 0 ; exit 1

ARG PUID=1000
ENV PUID=${PUID}
ARG PGID=1000
ENV PGID=${PGID}

RUN groupmod -o -g ${PGID} www-data && \
    usermod -o -u ${PUID} -g www-data www-data

RUN groupmod -o -g ${PGID} nginx && \
    usermod -o -u ${PUID} -g nginx nginx

COPY start-container /usr/local/bin/start-container
RUN chmod +x /usr/local/bin/start-container

# Setup document root
RUN mkdir -p /var/www/
RUN chown -R www-data:www-data /var/www/

# Add application
WORKDIR /var/www/

# Expose the port nginx is reachable on
EXPOSE 80

ENTRYPOINT ["start-container"]
