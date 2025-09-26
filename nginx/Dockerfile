# ================================
# 阶段 1：构建阶段 - 编译 Nginx（启用 stream 模块）
# ================================
FROM alpine:latest AS builder

# 设置版本变量
ENV NGINX_VERSION=1.26.2 \
    PREFIX=/usr/local/nginx

# 更换为阿里云镜像源（解决国内网络问题）
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# 安装完整编译依赖（关键：包含 linux-headers）
RUN apk add --no-cache \
    gcc \
    g++ \
    make \
    autoconf \
    libc-dev \
    linux-headers \
    pcre-dev \
    zlib-dev \
    openssl-dev \
    wget \
    tar \
    ca-certificates

# 下载并解压 Nginx 源码（禁用权限还原）
RUN echo "📥 下载 Nginx ${NGINX_VERSION}..." && \
    wget -q https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz && \
    echo "📦 解压中..." && \
    tar -xzf nginx-${NGINX_VERSION}.tar.gz --no-same-permissions -C / && \
    rm -f nginx-${NGINX_VERSION}.tar.gz && \
    echo "✅ 解压完成"

# 进入源码目录并配置
RUN echo "⚙️  配置 Nginx..." && \
    cd /nginx-${NGINX_VERSION} && \
    ./configure \
        --prefix=${PREFIX} \
        --with-stream \
        --with-stream_ssl_module \
        --with-http_ssl_module \
        --with-http_v2_module \
        --with-http_realip_module \
        --with-http_gzip_static_module \
        --with-http_stub_status_module \
        --with-threads \
        --with-file-aio \
        --with-compat \
        --without-http_uwsgi_module \
        --without-http_scgi_module \
        --without-http_fastcgi_module \
    && \
    echo "✅ 配置完成"

# 编译并安装
RUN echo "🔧 开始编译..." && \
    cd /nginx-${NGINX_VERSION} && \
    make -j$(nproc) && \
    make install && \
    echo "✅ 编译安装完成"

# 清理
RUN rm -rf /nginx-${NGINX_VERSION}


# ================================
# 阶段 2：运行阶段 - 构建轻量镜像
# ================================
FROM alpine:latest

LABEL maintainer="dev@example.com"
LABEL description="Nginx 1.26.2 with stream module, minimal & secure"

# 使用阿里云源
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# 安装运行时依赖
RUN apk add --no-cache \
    libgcc \
    openssl \
    ca-certificates

# 创建 Nginx 用户
RUN adduser -D -s /sbin/nologin nginx

# 复制编译好的 Nginx
COPY --from=builder ${PREFIX} ${PREFIX}

# 日志输出到 stdout/stderr
RUN ln -sf /dev/stdout ${PREFIX}/logs/access.log && \
    ln -sf /dev/stderr ${PREFIX}/logs/error.log

# 复制默认配置
RUN mkdir -p /etc/nginx && \
    cp -r ${PREFIX}/conf/* /etc/nginx/ && \
    mkdir -p /etc/nginx/conf.d

# 暴露端口
EXPOSE 80 443 3306 6379

# 挂载点
VOLUME ["/etc/nginx", "/usr/local/nginx/html", "/usr/local/nginx/logs"]

# 使用非 root 用户
USER nginx

# 启动命令
CMD ["/usr/local/nginx/sbin/nginx", "-g", "daemon off;"]
