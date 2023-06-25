FROM alpine:3.12

LABEL Maintainer="chuoke"

# mirrors.aliyun.com
# mirrors.tuna.tsinghua.edu.cn
ARG ALPINE_APK_REPO=mirrors.tuna.tsinghua.edu.cn
ENV ALPINE_APK_REPO=${ALPINE_APK_REPO}

RUN if [ ${ALPINE_APK_REPO} ]; then \
        sed -i "s/dl-cdn.alpinelinux.org/${ALPINE_APK_REPO}/g" /etc/apk/repositories; \
    fi && \
    apk update \
    && apk upgrade \
    && apk add --no-cache openssl bash curl ca-certificates shadow

# 修改时区
RUN apk add -U tzdata; \
    cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime; \
    apk del tzdata

## Install about PHP
RUN apk --no-cache add \
    php7 \
    php7-common \
    php7-fpm \
    php7-opcache \
    php7-bcmath \
    php7-ctype \
    php7-fileinfo \
    php7-json \
    php7-mbstring \
    php7-openssl \
    php7-pdo \
    php7-tokenizer \
    php7-xml \
    php7-curl \
    php7-dom \
    php7-gd \
    php7-simplexml \
    php7-xmlwriter \
    php7-xmlreader \
    php7-posix \
    php7-zlib \
    php7-zip \
    php7-sodium \
    php7-phar \
    php7-exif \
    php7-iconv \
    php7-intl \
    php7-session \
    php7-ftp \
    php7-sockets \
    php7-calendar \
    php7-mysqli \
    php7-pdo_mysql \
    php7-pdo_sqlite \
    php7-pcntl \
    php7-gettext \
    php7-redis \
    php7-uuid \
    php7-xdebug \
    php7-apcu \
    php7-imagick

# Configure PHP-FPM
COPY php-fpm.conf /etc/php7/php-fpm.d/www.conf
COPY php.ini /etc/php7/conf.d/custom.ini

RUN ln -s /usr/sbin/php-fpm7 /usr/sbin/php-fpm

# Install about nginx
RUN apk --no-cache add nginx && \
    rm /etc/nginx/conf.d/default.conf

# Configure nginx
COPY nginx.conf /etc/nginx/nginx.conf
COPY nginx-server-temps /etc/nginx/server-temps

# Install supervisor
RUN apk --no-cache add supervisor
# Configure supervisord
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Install composer
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

# RUN set -x ; \
#     addgroup -g 1000 -S www-data ; \
#     adduser -u 1000 -D -S -G www-data www-data && exit 0 ; exit 1

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

# Clean
RUN rm -rf /var/cache/apk/*

RUN mkdir -p /var/www/
RUN chown -R www-data:www-data /var/www/

WORKDIR /var/www/

# Expose the port nginx is reachable on
EXPOSE 80 443

COPY start-container /usr/local/bin/start-container
RUN chmod +x /usr/local/bin/start-container

ENTRYPOINT ["start-container"]

# Let supervisord start nginx & php-fpm
# CMD ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]
