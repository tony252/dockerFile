# ================================
# 阶段 1：构建阶段 - 编译 Nginx
# ================================
FROM alpine:latest AS builder

ENV NGINX_VERSION=1.26.2 \
    NJS_VERSION=0.8.6 \
    PREFIX=/usr/local/nginx

# 使用阿里云源
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# 安装编译依赖（静默）
RUN apk add --no-cache -q \
    gcc g++ make autoconf libc-dev \
    pcre-dev zlib-dev openssl-dev wget tar

# 下载并解压 Nginx 源码（禁用权限还原）
RUN wget -q https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz && \
    tar -xzf nginx-${NGINX_VERSION}.tar.gz --no-same-permissions -C / && \
    rm -f nginx-${NGINX_VERSION}.tar.gz

# 下载并解压 NJS 模块
RUN wget -q https://github.com/nginx/njs/releases/download/${NJS_VERSION}/njs-${NJS_VERSION}.tar.gz && \
    tar -xzf njs-${NJS_VERSION}.tar.gz --no-same-permissions -C / && \
    rm -f njs-${NJS_VERSION}.tar.gz

# 配置并编译
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
RUN rm -rf /nginx-${NGINX_VERSION} /njs-${NJS_VERSION}


# ================================
# 阶段 2：运行阶段 - 关键修复在此
# ================================
FROM alpine:latest

# 使用阿里云源
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# ✅ 修复：使用 openssl 代替 libssl1.1
RUN apk add --no-cache -q \
    libgcc \
    openssl \
    ca-certificates

# 创建用户
RUN adduser -D -s /sbin/nologin nginx

# 复制 Nginx
COPY --from=builder ${PREFIX} ${PREFIX}

# 日志输出到 stdout/stderr
RUN ln -sf /dev/stdout /usr/local/nginx/logs/access.log && \
    ln -sf /dev/stderr /usr/local/nginx/logs/error.log

# 复制配置
RUN mkdir -p /etc/nginx && \
    cp -r /usr/local/nginx/conf/* /etc/nginx/

EXPOSE 80 443 3306 6379

VOLUME ["/etc/nginx", "/usr/local/nginx/html", "/usr/local/nginx/logs"]

USER nginx

CMD ["/usr/local/nginx/sbin/nginx", "-g", "daemon off;"]
