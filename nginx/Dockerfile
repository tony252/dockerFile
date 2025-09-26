# ================================
# 阶段 1：构建阶段 - 编译 Nginx
# ================================
FROM alpine:latest AS builder

# 设置变量
ENV NGINX_VERSION=1.26.2 \
    PREFIX=/usr/local/nginx

# 更换为阿里云源
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# 安装编译依赖
RUN apk add --no-cache \
    gcc g++ make autoconf libc-dev linux-headers \
    pcre-dev zlib-dev openssl-dev wget tar

# 下载并解压 Nginx
RUN wget -q https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz && \
    tar -xzf nginx-${NGINX_VERSION}.tar.gz --no-same-permissions -C / && \
    rm -f nginx-${NGINX_VERSION}.tar.gz

# 配置并编译
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
    && \
    make -j$(nproc) && \
    make install

# 清理
RUN rm -rf /nginx-${NGINX_VERSION}

# ================================
# 阶段 2：运行阶段 - 修复 PCRE 依赖
# ================================
FROM alpine:latest

# ✅ 重新定义环境变量
ENV PREFIX=/usr/local/nginx

# 使用阿里云源
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# ✅ 安装运行时依赖：包含 pcre、openssl 等
RUN apk add --no-cache \
    libgcc \
    openssl \
    pcre \           # ✅ 关键：提供 libpcre.so.1
    ca-certificates

# 创建 Nginx 用户
RUN adduser -D -s /sbin/nologin nginx

# 复制编译好的 Nginx
COPY --from=builder ${PREFIX} ${PREFIX}

# 确保日志目录存在
RUN mkdir -p ${PREFIX}/logs

# 软链接日志到 stdout/stderr
RUN ln -sf /dev/stdout ${PREFIX}/logs/access.log && \
    ln -sf /dev/stderr ${PREFIX}/logs/error.log

# 复制配置文件
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
