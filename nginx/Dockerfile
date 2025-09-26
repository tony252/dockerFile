FROM nginx:latest

RUN apt-get update && apt-get install -y \
    build-essential \
    libpcre3-dev \
    zlib1g-dev \
    wget

WORKDIR /usr/src/nginx

RUN wget https://nginx.org/download/nginx-1.22.1.tar.gz \
    && tar -xzvf nginx-1.22.1.tar.gz

WORKDIR /usr/src/nginx/nginx-1.22.1

RUN ./configure --with-stream && make && make install

ADD nginx.conf /etc/nginx/nginx.conf

RUN ln -sf /dev/stdout /var/log/nginx/access.log && ln -sf /dev/stderr /var/log/nginx/error.log

EXPOSE 9888
STOPSIGNAL SIGTERM
CMD ["nginx","-g","daemon off;"]
