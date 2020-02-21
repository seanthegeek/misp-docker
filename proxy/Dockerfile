FROM nginx:1.16

ARG MISP_FQDN

RUN apt-get update

RUN apt-get install -y openssl ca-certificates

RUN mkdir -p /etc/nginx/ssl

RUN openssl req -new -newkey rsa:4096 -days 365 -nodes -x509 \
    -subj "/CN=$MISP_FQDN/O=Acme Corporation/OU=Information Security/C=US/ST=Denial/L=Springfield" \
    -keyout /etc/nginx/ssl/misp.key  \
    -out /etc/nginx/ssl/misp.crt

# Copy  default conf file for proxy service
COPY ./default.conf /etc/nginx/conf.d/default.conf

# Copy proxy SSL certificates
COPY ./ssl/ /etc/nginx/ssl/
