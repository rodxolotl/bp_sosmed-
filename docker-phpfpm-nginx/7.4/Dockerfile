FROM alpine:3.11

# make sure you can use HTTPS
RUN apk --update add ca-certificates

RUN echo "https://dl.bintray.com/php-alpine/v3.11/php-7.4" >> /etc/apk/repositories

# Install packages
RUN apk --no-cache add php php-fpm php-pdo php-mysqli php-opcache php-openssl php-curl php-pdo_mysql php-gd php-intl php-mbstring php-soap php-xml php-xmlrpc php-zip php-session \
    nginx supervisor curl vim

# Configure nginx
COPY config/nginx.conf /etc/nginx/nginx.conf

# Remove default server definition
RUN rm /etc/nginx/conf.d/default.conf

# Configure PHP-FPM
COPY config/fpm-pool.conf /etc/php7/php-fpm.d/www.conf
COPY config/php.ini /etc/php7/conf.d/custom.ini

# Configure supervisord
COPY config/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Setup document root
RUN mkdir -p /var/www/html

# Make sure files/folders needed by the processes are accessable when they run under the nobody user
RUN chown -R nobody.nobody /var/www/html && \
  chown -R nobody.nobody /run && \
  chown -R nobody.nobody /var/lib/nginx && \
  chown -R nobody.nobody /var/log/nginx

# Switch to use a non-root user from here on
USER nobody

# Add application
WORKDIR /var/www/html
COPY --chown=nobody src/sosial-media/ /var/www/html/
RUN chmod -R 755 /var/www/html

# Expose the port nginx is reachable on
EXPOSE 8080

# Let supervisord start nginx & php-fpm
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]

# Configure a healthcheck to validate that everything is up&running
HEALTHCHECK --timeout=10s CMD curl --silent --fail http://127.0.0.1:8080/fpm-ping
