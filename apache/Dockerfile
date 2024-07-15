ARG APACHE_VERSION=""
FROM httpd:${APACHE_VERSION:+${APACHE_VERSION}-}alpine

# https://qiita.com/keys/items/83379369f8ef8a9aaeec これによりinstallが少ない代わりに重い
RUN apk update; \
    apk upgrade;
  
RUN apk add bash;

# Copy apache vhost file to proxy php requests to php-fpm container
COPY local.apache.conf /usr/local/apache2/conf/local.apache.conf
RUN echo "Include /usr/local/apache2/conf/local.apache.conf" \
    >> /usr/local/apache2/conf/httpd.conf
    
