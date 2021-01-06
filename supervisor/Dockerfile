FROM python:3.9-alpine3.12

LABEL maintainer="lestat@lestat.me"

# 中科大镜像
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories

# 安装
RUN apk update \
    && apk add --no-cache supervisor

# 挂载配置
COPY supervisord.conf /etc/supervisord/supervisord.conf

WORKDIR /etc/supervisord

# 启动supervisord服务
CMD ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisord/supervisord.conf"]