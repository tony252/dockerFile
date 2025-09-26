# ================================
# 阶段 1：构建阶段 - 编译 Nginx（启用 stream 模块）
# ================================
FROM alpine:latest AS builder

LABEL maintainer="your-email@example.com"
LABEL description="Nginx 1.26.2 with stream module, built from source"

# 设置版本变量
ENV NGINX_VERSION=1.26.2 \
    NJS_VERSION=0.8.6 \
    PREFIX=/usr/local/nginx

# 更换为阿里云镜像源（解决国内网络问题）
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# 安装编译依赖
RUN apk add --no-cache --quiet \
    gcc \
    g++ \
    make \
    autoconf \
    libc-dev \
    pcre-dev \
    zlib-dev \
    openssl-dev \
    wget \
    tar

# 下载并解压 Nginx 源码（关键：禁用权限还原）
RUN wget -q https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz && \
    tar -zxvf nginx-${NGINX_VERSION}.tar.gz \
        --no-same-permissions \
        --warning=no-timestamp \
        --warning=no-unknown-keyword && \
    rm -f nginx-${NGINX_VERSION}.tar.gz

# 下载并解压 NJS 模块（可选）
RUN wget -q https://github.com/nginx/njs/releases/download/${NJS_VERSION}/njs-${NJS_VERSION}.tar.gz && \
    tar -zxvf njs-${NJS_VERSION}.tar.gz \
        --no-same-permissions \
        --warning=no-timestamp \
        --warning=no-unknown-keyword && \
    rm -f njs-${NJS_VERSION}.tar.gz

# 配置并编译 Nginx（启用 stream 和常用模块）
RUN cd nginx-${NGINX_VERSION} && \
    ./configure \
        --prefix=${PREFIX} \
        --with-stream \
        --with-stream_ssl_module \
        --with-stream_ssl_preread_module \
        --with-http_ssl_module \
        --with-http_v2_module \
        --with-http_realip_module \
        --with-http_gzip_static_module \
        --with-http_stub_status_module \
        --with-threads \
        --with-file-aio \
        --with-compat \
        --add-dynamic-module=../njs-${NJS_VERSION}/nginx \
    && \
    make -j$(nproc) && \
    make install

# 清理源码（减少中间层）
RUN rm -rf /nginx-${NGINX_VERSION} /njs-${NJS_VERSION}


# ================================
# 阶段 2：运行阶段 - 构建轻量镜像
# ================================
FROM alpine:latest

# 更换为阿里云镜像源
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# 安装运行时依赖（使用 openssl，兼容 Alpine 3.18+）
# 注意：libssl1.1 已被移除，应使用 openssl 或 libssl3
RUN apk add --no-cache --quiet \
    libgcc \
    openssl \
    ca-certificates

# 创建 Nginx 用户
RUN adduser -D -s /sbin/nologin nginx

# 复制编译好的 Nginx
COPY --from=builder ${PREFIX} ${PREFIX}

# 创建日志软链接，输出到 stdout/stderr
RUN ln -sf /dev/stdout /usr/local/nginx/logs/access.log && \
    ln -sf /dev/stderr /usr/local/nginx/logs/error.log

# 创建配置目录并复制默认配置
RUN mkdir -p /etc/nginx && \
    cp -r /usr/local/nginx/conf/* /etc/nginx/

# 暴露端口
EXPOSE 80 443 3306 6379

# 挂载点（便于外部管理）
VOLUME ["/etc/nginx", "/usr/local/nginx/html", "/usr/local/nginx/logs"]

# 使用非 root 用户运行
USER nginx

# 启动命令
CMD ["/usr/local/nginx/sbin/nginx", "-g", "daemon off;"]
