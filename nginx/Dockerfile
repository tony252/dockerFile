# ================================
# 阶段 1：构建阶段 - 修复 njs 下载（使用 codeload）
# ================================
FROM alpine:latest AS builder

ENV NGINX_VERSION=1.26.2 \
    NJS_VERSION=0.8.6 \
    PREFIX=/usr/local/nginx

# 使用阿里云源
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# 安装依赖
RUN apk add --no-cache -q \
    gcc g++ make autoconf libc-dev \
    pcre-dev zlib-dev openssl-dev wget tar

# 下载并解压 Nginx
RUN wget -q https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz && \
    tar -xzf nginx-${NGINX_VERSION}.tar.gz --no-same-permissions -C / && \
    rm -f nginx-${NGINX_VERSION}.tar.gz

# ✅ 使用正确的 codeload 链接下载 njs
RUN wget -q https://codeload.github.com/nginx/njs/tar.gz/refs/tags/${NJS_VERSION} -O njs-${NJS_VERSION}.tar.gz && \
    tar -xzf njs-${NJS_VERSION}.tar.gz --no-same-permissions -C / && \
    mv /njs-refs-tags-${NJS_VERSION} /njs-${NJS_VERSION} && \
    rm -f njs-${NJS_VERSION}.tar.gz
