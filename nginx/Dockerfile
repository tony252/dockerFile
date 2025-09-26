# ================================
# 阶段 1：构建阶段 - 修复 njs 下载
# ================================
FROM alpine:latest AS builder

ENV NGINX_VERSION=1.26.2 \
    NJS_VERSION=0.8.6 \
    PREFIX=/usr/local/nginx

# 使用阿里云源
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# 安装编译依赖
RUN apk add --no-cache -q \
    gcc g++ make autoconf libc-dev \
    pcre-dev zlib-dev openssl-dev wget tar

# 下载并解压 Nginx
RUN wget -q https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz && \
    tar -xzf nginx-${NGINX_VERSION}.tar.gz --no-same-permissions -C / && \
    rm -f nginx-${NGINX_VERSION}.tar.gz

# ✅ 修复：使用正确的 njs 下载链接
RUN wget -q https://github.com/nginx/njs/archive/refs/tags/${NJS_VERSION}.tar.gz && \
    tar -xzf ${NJS_VERSION}.tar.gz --no-same-permissions -C / && \
    mv /${NJS_VERSION} /njs-${NJS_VERSION} && \
    rm -f ${NJS_VERSION}.tar.gz

# 配置并编译 Nginx（注意路径）
RUN cd /nginx-${NGINX_VERSION} && \
    ./configure \
        --prefix=${PREFIX} \
        --with-stream \
        --with-stream_ssl_module \
        --with-http_ssl_module \
        --with-http_v2_module \
        --with-threads \
        --with-compat \
        --add-dynamic-module=/njs-${NJS_VERSION}/nginx \
    > /dev/null && \
    make -s -j$(nproc) && \
    make install > /dev/null

# 清理
RUN rm -rf /nginx-${NGINX_VERSION}
