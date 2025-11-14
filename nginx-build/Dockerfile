FROM openresty/openresty:alpine

# copy configs
COPY nginx.conf /etc/nginx/nginx.conf
COPY wordpress-proxy.conf /etc/nginx/conf.d/wordpress-proxy.conf

# create log dir
RUN mkdir -p /var/log/nginx

EXPOSE 80

CMD ["openresty", "-g", "daemon off;"]
