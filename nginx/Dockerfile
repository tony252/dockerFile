# ================================
# 阶段 1：构建阶段
# ================================
FROM alpine:latest AS builder

ENV NGINX_VERSION=1.26.2 \
    PREFIX=/usr/local/nginx

# 使用阿里云源（国内加速）
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# 安装编译依赖（包含 pcre-dev）
RUN apk add --no-cache \
    gcc g++ make autoconf libc-dev linux-headers \
    pcre-dev zlib-dev openssl-dev wget tar

# 下载并编译 Nginx
RUN wget -q https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz && \
    tar -xzf nginx-${NGINX_VERSION}.tar.gz -C / && \
    rm -f nginx-${NGINX_VERSION}.tar.gz

RUN cd /nginx-${NGINX_VERSION} && \
    ./configure \
        --prefix=${PREFIX} \
        --with-stream \
        --with-stream_ssl_module \
        --with-http_ssl_module \
        --with-http_v2_module \
        --with-http_gzip_static_module \
        --with-threads \
        --with-file-aio \
        --with-compat \
    && make -j$(nproc) && make install

RUN rm -rf /nginx-${NGINX_VERSION}


# ================================
# 阶段 2：运行阶段（✅ 关键：必须安装 pcre）
# ================================
FROM alpine:latest

# ✅ 重新定义变量
ENV PREFIX=/usr/local/nginx

# 使用阿里云源
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# ✅ 安装运行时依赖：pcre 是必须的！
RUN apk add --no-cache \
    libgcc \
    openssl \
    pcre \              # <<<--- 这是解决 libpcre.so.1 问题的关键
    ca-certificates

# 创建用户
RUN adduser -D -s /sbin/nologin nginx

# 复制 Nginx
COPY --from=builder ${PREFIX} ${PREFIX}

# 创建日志目录
RUN mkdir -p ${PREFIX}/logs

# 日志重定向
RUN ln -sf /dev/stdout ${PREFIX}/logs/access.log && \
    ln -sf /dev/stderr ${PREFIX}/logs/error.log

# 配置文件
RUN mkdir -p /etc/nginx && \
    cp -r ${PREFIX}/conf/* /etc/nginx/ && \
    mkdir -p /etc/nginx/conf.d

EXPOSE 80 443

VOLUME ["/etc/nginx", "/usr/local/nginx/html", "/usr/local/nginx/logs"]

USER nginx

CMD ["/usr/local/nginx/sbin/nginx", "-g", "daemon off;"]
