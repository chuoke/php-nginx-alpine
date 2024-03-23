FROM alpine:3.18

LABEL Maintainer="chuoke"

# mirrors.aliyun.com
# mirrors.tuna.tsinghua.edu.cn
ARG ALPINE_APK_REPO=mirrors.aliyun.com
ENV ALPINE_APK_REPO=${ALPINE_APK_REPO}

# RUN apk upgrade
# RUN apk add -X https://dl-cdn.alpinelinux.org/alpine/v3.17/main -u alpine-keys

# swoole 在 testing 库
RUN echo "https://dl-cdn.alpinelinux.org/alpine/edge/testing/" >> /etc/apk/repositories

RUN if [ ${ALPINE_APK_REPO} ]; then \
        sed -i "s/dl-cdn.alpinelinux.org/${ALPINE_APK_REPO}/g" /etc/apk/repositories; \
    fi

# RUN apk update \
#     && apk upgrade \
RUN apk add --no-cache openssl bash curl ca-certificates shadow \
    nodejs

## Install about PHP

RUN apk add --no-cache \
    php81 \
    php81-common \
    php81-fpm \
    # Laravel Requirements
    php81-bcmath \
    php81-ctype \
    php81-fileinfo \
    php81-json \
    php81-mbstring \
    php81-openssl \
    php81-pdo \
    php81-tokenizer \
    php81-xml \
    # Base Requirements
    php81-curl \
    php81-dom \
    php81-gd \
    php81-opcache \
    php81-xmlwriter \
    php81-simplexml \
    php81-posix \
    php81-opcache \
    # Other Requirements
    php81-zlib \
    php81-zip \
    php81-sodium \
    php81-phar \
    php81-exif \
    php81-iconv \
    php81-intl \
    php81-session \
    php81-xmlreader \
    php81-ftp \
    php81-sockets \
    php81-calendar \
    php81-mysqli \
    php81-pdo_mysql \
    php81-pdo_sqlite \
    php81-sqlite3 \
    php81-pcntl \
    php81-gettext \
    php81-imap \
    # pecl extensions
    php81-pecl-apcu \
    php81-pecl-redis \
    php81-pecl-mongodb \
    php81-pecl-imagick \
    php81-pecl-xdebug \
    php81-pecl-swoole

# Configure PHP-FPM
COPY php-fpm.conf /etc/php81/php-fpm.d/www.conf
COPY php.ini /etc/php81/conf.d/custom.ini

RUN rm -f /usr/bin/php && ln -s /usr/bin/php81 /usr/bin/php && \
    ln -s /usr/sbin/php-fpm81 /usr/sbin/php-fpm

# Install composer
# RUN apk add composer # This way make size bigger
ENV COMPOSER_HOME /composer
ENV PATH ./vendor/bin:/composer/vendor/bin:$PATH
ENV COMPOSER_ALLOW_SUPERUSER 1
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" && \
    php composer-setup.php --install-dir=/usr/local/bin --filename=composer && \
    php -r "unlink('composer-setup.php');"

# Config Composer packagist repo
# ARG COMPOSER_REPO_PACKAGIST=https://mirrors.aliyun.com/composer/
ARG COMPOSER_REPO_PACKAGIST=https://mirrors.tencent.com/composer/
ENV COMPOSER_REPO_PACKAGIST=${COMPOSER_REPO_PACKAGIST}
RUN if [ ${COMPOSER_REPO_PACKAGIST} ]; then \
    composer config -g repo.packagist composer ${COMPOSER_REPO_PACKAGIST} \
;fi


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

# Clean
RUN rm -rf /var/cache/apk/*

# ensure www-data user exists
RUN set -x ; \
  addgroup -g 82 -S www-data ; \
  adduser -u 82 -D -S -G www-data www-data && exit 0 ; exit 1

ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}

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

# CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]

ENTRYPOINT ["start-container"]
