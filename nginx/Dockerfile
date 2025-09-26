# ================================
# 阶段 1：构建阶段 - 编译 Nginx（启用 stream 模块）
# ================================
FROM alpine:latest AS builder

# 设置版本变量
ENV NGINX_VERSION=1.26.2 \
    PREFIX=/usr/local/nginx

# 更换为阿里云镜像源（解决国内网络问题）
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# 安装编译依赖（包含 linux-headers）
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
    tar

# 下载并解压 Nginx 源码
RUN echo "📥 下载 Nginx ${NGINX_VERSION}..." && \
    wget -q https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz && \
    echo "📦 解压中..." && \
    tar -xzf nginx-${NGINX_VERSION}.tar.gz --no-same-permissions -C / && \
    rm -f nginx-${NGINX_VERSION}.tar.gz

# 配置并编译 Nginx
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
        --with-threads \
        --with-file-aio \
        --with-compat \
    && \
    echo "✅ 配置完成" && \
    make -j$(nproc) && \
    make install && \
    echo "✅ 编译安装完成"

# 清理源码
RUN rm -rf /nginx-${NGINX_VERSION}


# ================================
# 阶段 2：运行阶段 - 构建轻量镜像
# ================================
FROM alpine:latest

# ✅ 重要：重新定义环境变量（多阶段构建不会自动继承）
ENV PREFIX=/usr/local/nginx

# 使用阿里云镜像源
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# ✅ 安装运行时依赖（关键：包含 pcre 提供 libpcre.so.1）
RUN apk add --no-cache \
    libgcc \
    openssl \
    pcre \
    ca-certificates

# 创建 Nginx 用户（非 root）
RUN adduser -D -s /sbin/nologin nginx

# 复制编译好的 Nginx
COPY --from=builder ${PREFIX} ${PREFIX}

# ✅ 确保日志目录存在
RUN mkdir -p ${PREFIX}/logs

# 将日志重定向到 stdout/stderr（便于 docker logs 查看）
RUN ln -sf /dev/stdout ${PREFIX}/logs/access.log && \
    ln -sf /dev/stderr ${PREFIX}/logs/error.log

# 创建配置目录并复制默认配置
RUN mkdir -p /etc/nginx && \
    cp -r ${PREFIX}/conf/* /etc/nginx/ && \
    mkdir -p /etc/nginx/conf.d

# 暴露常用端口
EXPOSE 80 443 3306 6379

# 挂载点（便于外部管理配置、静态文件、日志）
VOLUME ["/etc/nginx", "/usr/local/nginx/html", "/usr/local/nginx/logs"]

# 使用非 root 用户运行
USER nginx

# 启动命令
CMD ["/usr/local/nginx/sbin/nginx", "-g", "daemon off;"]
