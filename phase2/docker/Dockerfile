FROM openresty/openresty:1.25.3.1-1-alpine

COPY default.conf /etc/nginx/conf.d/default.conf

EXPOSE 82

CMD ["/usr/local/openresty/bin/openresty", "-g", "daemon off;"]
