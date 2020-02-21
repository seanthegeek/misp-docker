FROM nginx:1.16

RUN apt-get update

RUN apt-get install -y openssl

RUN openssl dhparam -out /etc/ssl/certs/dhparam.pem 4096

RUN openssl req -new -newkey rsa:4096 -days 365 -nodes -x509 \
    -subj "/C=US/ST=Denial/L=Springfield/O=Dis/CN=${MISP_DOMAIN}" \
    -keyout ./ssl/ /etc/ssl/certs/nginx/misp.key  \
    -out ./ssl/ /etc/ssl/certs/nginx/misp.cert

#  default conf for proxy service
COPY ./default.conf /etc/nginx/conf.d/default.conf

#  Proxy and SSL configurations
COPY ./includes/ /etc/nginx/includes/

# Proxy SSL certificates
COPY ./ssl/ /etc/ssl/certs/nginx/
