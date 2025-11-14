FROM ubuntu:22.04

RUN apt-get update && apt-get install -y \
    build-essential \
    libpcre3 \
    libpcre3-dev \
    zlib1g-dev \
    libssl-dev \
    libgeoip-dev \
    libpq-dev \
    wget \
    git \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /tmp
RUN wget https://openresty.org/download/openresty-1.21.4.1.tar.gz \
    && tar -xzf openresty-1.21.4.1.tar.gz \
    && cd openresty-1.21.4.1 \
    && ./configure --prefix=/opt/openresty \
        --with-pcre-jit \
        --with-ipv6 \
        --without-http_redis2_module \
        --with-http_iconv_module \
        --with-http_postgres_module \
    && make -j8 \
    && make install \
    && cd .. \
    && rm -rf openresty-1.21.4.1*

RUN useradd -r -s /bin/false nginx

COPY nginx.conf /opt/openresty/nginx/conf/nginx.conf
COPY wordpress-proxy.conf /opt/openresty/nginx/conf/conf.d/wordpress-proxy.conf

RUN mkdir -p /var/log/nginx /var/cache/nginx \
    && chown -R nginx:nginx /var/log/nginx /var/cache/nginx

EXPOSE 80 443

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD ["/opt/openresty/nginx/sbin/nginx", "-t"]

CMD ["/opt/openresty/nginx/sbin/nginx", "-g", "daemon off;"]
