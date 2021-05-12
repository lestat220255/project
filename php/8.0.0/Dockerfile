FROM php:8.0.0-fpm-alpine3.12

# 修改镜像源
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

ADD imagick/install-php-extensions /usr/local/bin/install-php-extensions

RUN chmod uga+x /usr/local/bin/install-php-extensions && sync && \
    install-php-extensions imagick

# Extract PHP source
RUN apk update && apk add rabbitmq-c-dev \ 
&& docker-php-source extract \
# Create directory for amqp extension
&& mkdir /usr/src/php/ext/amqp \
# Download AMQP master branch files to extension directory
&& curl -L https://github.com/php-amqp/php-amqp/archive/master.tar.gz | tar -xzC /usr/src/php/ext/amqp --strip-components=1 \
# Install amqp extension using built-in docker binary
&& docker-php-ext-install amqp \
# Delete PHP source
&& docker-php-source delete

# 安装依赖,核心扩展,pecl扩展,composer工具
RUN apk update && apk add --no-cache --virtual .build-deps \
    $PHPIZE_DEPS \
    nginx \
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
    nginx \
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
    && docker-php-ext-install -j"$(getconf _NPROCESSORS_ONLN)" iconv opcache \
    && pecl install -o -f redis \
    && rm -rf /tmp/pear \
    && docker-php-ext-enable \
    redis

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

# 初始化nginx pid目录
RUN mkdir -p /run/nginx \
    && chmod -R 777 /run/nginx

WORKDIR /var/www/html

COPY supervisord.conf /etc/supervisord.conf

# start php-fpm via supervisor
ENTRYPOINT ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisord.conf"]