FROM nginx:latest

# Nginx will listen on this port
EXPOSE $NGINX_PORT

COPY ./templates /etc/nginx/templates

# Install certbot
RUN apt-get -y update
RUN apt-get -y install certbot python-certbot-nginx

COPY certbot.sh /docker-entrypoint.d/
RUN chmod +x /docker-entrypoint.d/certbot.sh

COPY certbot-renew /etc/cron.d/
RUN chmod +x /etc/cron.d/certbot-renew
