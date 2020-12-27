FROM nginx:latest

# Nginx will listen on this port
EXPOSE 80

# Remove the default config file that
# /etc/nginx/nginx.conf includes
RUN rm /etc/nginx/conf.d/default.conf

# Install certbot
#RUN apt-get -y update
#RUN apt-get -y install certbot python-certbot-nginx

#COPY docker-entrypoint.sh /usr/local/bin/
#RUN chmod +x /usr/local/bin/docker-entrypoint.sh

#COPY certbot-renew /etc/cron.d/
#RUN chmod +x /etc/cron.d/certbot-renew

COPY --from=cupldeploy_cuplfrontend /app/build /usr/share/nginx/html/cuplfrontend

COPY wsbackend.conf /etc/nginx/conf.d
#COPY wsfrontend.conf /etc/nginx/conf.d

#ENTRYPOINT ["docker-entrypoint.sh"]
