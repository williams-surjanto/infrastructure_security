FROM nginx:stable-trixie-perl

COPY nginx.conf /etc/nginx/nginx.conf
COPY default.conf /etc/nginx/conf.d/default.conf
COPY nginx-selfsigned.crt /etc/nginx/ssl/nginx-selfsigned.crt
COPY nginx-selfsigned.key /etc/nginx/ssl/nginx-selfsigned.key

RUN chmod 644 /etc/nginx/ssl/nginx-selfsigned.crt && chmod 600 /etc/nginx/ssl/nginx-selfsigned.key
