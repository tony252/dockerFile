FROM alpine:latest as builder
 
# 安装构建依赖
RUN apk add --no-cache gcc libc-dev make openssl-dev pcre-dev zlib-dev
 
# 获取nginx源代码
RUN wget http://nginx.org/download/nginx-1.21.3.tar.gz && \
    tar -zxvf nginx-1.21.3.tar.gz && \
    cd nginx-1.21.3 && \
    ./configure --with-stream --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp && \
    make && make install
 
# 创建运行时的镜像
FROM alpine:latest
COPY --from=builder /etc/nginx /etc/nginx
COPY --from=builder /usr/sbin/nginx /usr/sbin/nginx
COPY --from=builder /usr/lib/nginx /usr/lib/nginx
COPY nginx.conf /etc/nginx/nginx.conf
 
EXPOSE 80 443
CMD ["nginx", "-g", "daemon off;"]
