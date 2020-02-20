FROM nginx:1.9

RUN openssl dhparam -out /etc/ssl/certs/dhparam.pem 4096

#  default conf for proxy service
COPY ./default.conf /etc/nginx/conf.d/default.conf

#  Proxy and SSL configurations
COPY ./includes/ /etc/nginx/includes/

# Proxy SSL certificates
COPY ./ssl/ /etc/ssl/certs/nginx/
