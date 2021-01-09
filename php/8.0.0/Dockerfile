FROM php:8.0.0-fpm-alpine3.12

LABEL maintainer="lestat@lestat.me"

# 修改镜像源
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# 安装依赖,核心扩展,pecl扩展,composer工具
RUN apk update && apk add --no-cache --virtual .build-deps \
    $PHPIZE_DEPS \
    libsodium \
    curl-dev \
    imagemagick-dev \
    libtool \
    libxml2-dev \
    sqlite-dev \
    freetype-dev \
    libjpeg-turbo-dev \
    libpng-dev \
    curl \
    imagemagick \
    mysql-client \
    php7-bcmath \
    gettext \
    supervisor \
    && docker-php-ext-install \
    curl \
    pdo \
    pdo_mysql \
    pdo_sqlite \
    pcntl \
    tokenizer \
    xml \
    bcmath \
    mysqli \
    shmop \
    soap \
    sockets \
    sysvsem \
    && docker-php-ext-install -j"$(getconf _NPROCESSORS_ONLN)" iconv \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j"$(getconf _NPROCESSORS_ONLN)" gd \
    && pecl install -o -f redis \
    && rm -rf /tmp/pear \
    && docker-php-ext-enable redis

COPY imagick /imagick

RUN cd /imagick \
    && phpize && ./configure \
    && make \
    && make install \
    && rm -rf /imagick

# 安装composer并允许root用户运行
ENV COMPOSER_ALLOW_SUPERUSER=1
ENV COMPOSER_NO_INTERACTION=1
ENV COMPOSER_HOME=/usr/local/share/composer
RUN mkdir -p /usr/local/share/composer \
    && curl -o /tmp/composer-setup.php https://getcomposer.org/installer \
    && php /tmp/composer-setup.php --no-ansi --install-dir=/usr/local/bin --filename=composer --snapshot \
    && rm -f /tmp/composer-setup.* \
    # 配置composer中国全量镜像
    && composer config -g repo.packagist composer https://packagist.phpcomposer.com

# 设置时区
RUN apk add tzdata && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo "Asia/Shanghai" > /etc/timezone \
    && apk del tzdata

WORKDIR /var/www/html

COPY supervisord.conf /etc/supervisord.conf

# 对其他容器开放9000端口
EXPOSE 9000

# start php-fpm via supervisor
ENTRYPOINT ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisord.conf"]