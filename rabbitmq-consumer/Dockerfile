FROM php:7.4.30-cli

ENV DEBIAN_FRONTEND noninteractive
ENV TERM            xterm-color

ARG DEV_MODE
ENV DEV_MODE $DEV_MODE

#配置时区
ENV TZ=Asia/Shanghai
RUN echo $TZ > /etc/timezone
RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

COPY ./rootfilesystem/ /

RUN \
    set -ex && \
    apt-get update && \
    apt-get install -y curl unzip && \
    curl -sfL https://getcomposer.org/installer | php -- --install-dir=/usr/bin --filename=composer && \
    chmod +x /usr/bin/composer                                                                     && \
    composer self-update 2.2.18                                                    && \
    apt-get update              && \
    apt-get install -y             \
        libcurl4-openssl-dev       \
        libssl-dev                 \
        supervisor                 \
        unzip                      \
        zlib1g-dev                 \
        procps                     \
        --no-install-recommends && \
    docker-php-ext-install pdo_mysql && \
    pecl channel-update pecl.php.net && \
    pecl install --configureoptions 'enable-redis-igbinary="yes" enable-redis-lzf="no" enable-redis-zstd="no"' redis-5.3.7 && \
    docker-php-ext-enable redis && \
    install-swoole.sh 4.8.12 \
        --enable-http2   \
        --enable-mysqlnd \
        --enable-openssl \
        --enable-sockets --enable-swoole-curl --enable-swoole-json && \
    mkdir -p /var/log/supervisor && \
    rm -rf /var/lib/apt/lists/* $HOME/.composer/*-old.phar /usr/bin/qemu-*-static


# 安装 Consul 和 Confd
RUN apt-get update \
    && apt-get install -y curl unzip \
    && curl -LO https://releases.hashicorp.com/consul/1.9.3/consul_1.9.3_linux_amd64.zip \
    && unzip consul_1.9.3_linux_amd64.zip \
    && rm consul_1.9.3_linux_amd64.zip \
    && mv consul /usr/local/bin/ \
    && curl -LO https://github.com/kelseyhightower/confd/releases/download/v0.16.0/confd-0.16.0-linux-amd64 \
    && chmod +x confd-0.16.0-linux-amd64 \
    && mv confd-0.16.0-linux-amd64 /usr/local/bin/confd \
    && mkdir /etc/consul.d \
    && mkdir -p /var/log/consul \
    && apt-get remove -y curl unzip \
    && apt-get autoremove -y \
    && rm -rf /var/lib/apt/lists/*


RUN mkdir -p /etc/confd/conf.d /etc/confd/templates /log/graylog /data/consul
COPY ./confd/service/supervisor /etc/confd/templates/supervisor.tmpl
COPY ./confd/service/config.toml /etc/confd/conf.d/

RUN chmod 777 /var/run \
    && chmod 777 /var/log

ENTRYPOINT ["/entrypoint.sh"]
CMD []

EXPOSE 8500 9001

WORKDIR "/var/www/"
