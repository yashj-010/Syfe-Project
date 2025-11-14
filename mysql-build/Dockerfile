FROM mysql:8.0

# copy initialization SQL
COPY init-wordpress.sql /docker-entrypoint-initdb.d/init-wordpress.sql

# set default charset and configuration via configmap in k8s
