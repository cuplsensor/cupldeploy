FROM nginx:latest

# Nginx will listen on this port
EXPOSE 80

# Remove the default config file that
# /etc/nginx/nginx.conf includes
RUN rm /etc/nginx/conf.d/default.conf

# Install certbot
RUN apt-get -y update
RUN apt-get -y install certbot python-certbot-nginx

COPY docker-entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-entrypoint.sh

COPY certbot-renew /etc/cron.d/
RUN chmod +x /etc/cron.d/certbot-renew

#COPY --from=wsfrontend_f3 /app/build /usr/share/nginx/html/wsfrontend_f3

#COPY wsbackend_b3.conf /etc/nginx/conf.d
#COPY wsfrontend_f3.conf /etc/nginx/conf.d

ENTRYPOINT ["docker-entrypoint.sh"]
