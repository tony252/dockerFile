ARG BASE_IMAGE=ubuntu:jammy
FROM ${BASE_IMAGE} as builder

ARG NGINX_VERSION=1.26.1
ENV DEBIAND_FRONTEND=noninteractive

RUN apt-get update \
 && apt-get install -y wget build-essential libpcre3 libpcre3-dev zlib1g zlib1g-dev libssl-dev libgd-dev libxml2 libxml2-dev uuid-dev unzip

RUN wget https://nginx.org/download/nginx-$NGINX_VERSION.tar.gz \
 && tar xvfz nginx-$NGINX_VERSION.tar.gz \
 && rm -f nginx-$NGINX_VERSION.tar.get \
 && mv nginx-$NGINX_VERSION nginx \
 && wget -O nginx_upstream_check_module-master.zip  https://codeload.github.com/yaoweibin/nginx_upstream_check_module/zip/refs/heads/master \
 && unzip nginx_upstream_check_module-master.zip  \
 && mv nginx_upstream_check_module-master /nginx_upstream_check_module

RUN cd /nginx \
 && patch -p1 < /nginx_upstream_check_module/check_1.20.1+.patch \
 && ./configure \
    --prefix=/etc/nginx \
    --sbin-path=/usr/sbin/nginx \
    --modules-path=/etc/nginx/modules \
    --conf-path=/etc/nginx/nginx.conf \
    --error-log-path=/var/log/nginx/error.log 
