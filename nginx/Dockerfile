# ================================
# 阶段 1：构建阶段 - 编译 Nginx（启用 stream 模块）
# ================================
FROM alpine:latest AS builder

# 设置版本变量
ENV NGINX_VERSION=1.26.2 \
    PREFIX=/usr/local/nginx

# 更换为阿里云镜像源（解决国内网络问题）
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# 安装编译依赖（静默）
RUN apk add --no-cache -q \
    gcc g++ make autoconf libc-dev \
    pcre-dev zlib-dev openssl-dev wget tar

# 下载并解压 Nginx 源码（禁用权限还原）
RUN wget -q https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz && \
    tar -xzf nginx-${NGINX_VERSION}.tar.gz --no-same-permissions -C / && \
    rm -f nginx-${NGINX_VERSION}.tar.gz

# 配置并编译 Nginx（启用 stream、SSL、HTTP/2 等常用模块）
RUN cd /nginx-${NGINX_VERSION} && \
    ./configure \
        --prefix=${PREFIX} \
        --with-stream \
        --with-stream_ssl_module \
        --with-http_ssl_module \
        --with-http_v2_module \
        --with-http_realip_module \
        --with-http_gzip_static_module \
        --with-threads \
        --with-file-aio \
        --with-compat \
    > /dev/null && \
    make -s -j$(nproc) && \
    make install > /dev/null

# 清理源码
RUN rm -rf /nginx-${NGINX_VERSION}


# ================================
# 阶段 2：运行阶段 - 构建轻量镜像
# ================================
FROM alpine:latest

# 使用阿里云镜像源
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# 安装运行时依赖（✅ 使用 openssl，兼容 Alpine 3.18+）
RUN apk add --no-cache -q \
    libgcc \
    openssl \
    ca-certificates

# 创建 Nginx 用户（非 root）
RUN adduser -D -s /sbin/nologin nginx

# 复制编译好的 Nginx
COPY --from=builder ${PREFIX} ${PREFIX}

# 日志输出到 stdout/stderr
RUN ln -sf /dev/stdout /usr/local/nginx/logs/access.log && \
    ln -sf /dev/stderr /usr/local/nginx/logs/error.log

# 创建配置目录并复制默认配置
RUN mkdir -p /etc/nginx && \
    cp -r /usr/local/nginx/conf/* /etc/nginx/

# 暴露端口
EXPOSE 80 443

# 挂载点（便于外部管理）
VOLUME ["/etc/nginx", "/usr/local/nginx/html", "/usr/local/nginx/logs"]

# 使用非 root 用户运行
USER nginx

# 启动命令
CMD ["/usr/local/nginx/sbin/nginx", "-g", "daemon off;"]
