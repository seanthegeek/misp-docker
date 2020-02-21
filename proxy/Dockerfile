FROM nginx:1.16

RUN apt-get update

RUN apt-get install -y openssl ca-certificates

RUN openssl req -new -newkey rsa:4096 -days 365 -nodes -x509 \
    -subj "/C=US/ST=Denial/L=Springfield/O=Dis/CN=$MISP_DOMAIN" \
    -keyout /etc/ssl/certs/nginx/misp.key  \
    -out /etc/ssl/certs/nginx/misp.cert

# Copy  default conf file for proxy service
COPY ./default.conf /etc/nginx/conf.d/default.conf

# Copy roxy SSL certificates
COPY ./ssl/ /etc/ssl/certs/nginx/
