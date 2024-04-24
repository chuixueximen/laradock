FROM laradock_workspace:latest

# config env
ENV FMS_ENV=local

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

# 安装 Supervisor
RUN apt-get update && apt-get install -y supervisor && \
    mkdir -p /var/log/supervisor && \
    touch /var/log/supervisor/supervisord.log && \
    chmod 777 /var/log/supervisor/supervisord.log

# 复制 Supervisor 配置文件
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
RUN mkdir -p /etc/confd/conf.d /etc/confd/templates /log/graylog
COPY ./confd/service/supervisor /etc/confd/templates/supervisor.tmpl
COPY ./confd/service/config.toml /etc/confd/conf.d/

# sync code
RUN mkdir -p /fms-mq

RUN mkdir -p /var/log/fmsmq /data/consul
RUN chmod -R 777 /tmp/ /var/log/fmsmq/

EXPOSE 8500

WORKDIR /fms-mq/

# 启动 Consul 和 Confd
CMD ["supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
