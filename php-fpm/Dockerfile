FROM daocloud.io/php:7.2-fpm-alpine

LABEL maintainer="lestat@lestat.me"

# 修改镜像源
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# 安装依赖,核心扩展,pecl扩展,git,composer,npm工具
RUN apk update && apk add --no-cache --virtual .build-deps \
        $PHPIZE_DEPS \
        curl-dev \
        imagemagick-dev \
        libtool \
        libxml2-dev \
		openssh-server \
		vim \
        postgresql-dev \
        sqlite-dev \
	    libmcrypt-dev \
        freetype-dev \
        libjpeg-turbo-dev \
        libpng-dev \
    && apk add --no-cache \
        curl \
        git \
        imagemagick \
        mysql-client \
        postgresql-libs \
        nodejs \
        nodejs-npm \
    # 配置npm中国镜像
    && npm config set registry https://registry.npm.taobao.org \
    && pecl install imagick \
    && pecl install mcrypt-1.0.1 \
    && docker-php-ext-enable mcrypt \
    && docker-php-ext-enable imagick \
    && docker-php-ext-install \
        curl \
        mbstring \
		sockets \
        bcmath \
        pdo \
        pdo_mysql \
        mysqli \
        pdo_pgsql \
        pdo_sqlite \
        pcntl \
        tokenizer \
        xml \
        zip \
	&& docker-php-ext-install -j"$(getconf _NPROCESSORS_ONLN)" iconv \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j"$(getconf _NPROCESSORS_ONLN)" gd \
    && pecl install -o -f redis \
    && pecl install -o -f swoole \
    && pecl install -o -f yar \
    && rm -rf /tmp/pear \
    && docker-php-ext-enable redis \
    && docker-php-ext-enable swoole \
    && docker-php-ext-enable yar

# 安装composer并允许root用户运行
ENV COMPOSER_ALLOW_SUPERUSER=1
ENV COMPOSER_NO_INTERACTION=1
ENV COMPOSER_HOME=/usr/local/share/composer
RUN mkdir -p /usr/local/share/composer \
	&& curl -o /tmp/composer-setup.php https://getcomposer.org/installer \
	&& php /tmp/composer-setup.php --no-ansi --install-dir=/usr/local/bin --filename=composer --snapshot \
	&& rm -f /tmp/composer-setup.* \
    # 配置composer中国全量镜像
    && composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/

#安装SSH服务
RUN mkdir -p /var/run/sshd
RUN mkdir -p /root/.ssh

#设置ssh远程登陆密码
RUN sed -i "s/#PermitRootLogin.*/PermitRootLogin yes/g" /etc/ssh/sshd_config && \
	mkdir -p /root/.ssh && chmod 700 /root/.ssh/ && \
    ssh-keygen -A && \
    echo "root:root" | chpasswd

#设置自启动命令
#ENTRYPOINT  ["/usr/sbin/sshd"]
#CMD ["/bin/bash", "/usr/sbin/sshd"]