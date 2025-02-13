FROM nginxinc/nginx-unprivileged:1.25.3

RUN rm /etc/nginx/conf.d/default.conf
COPY ./docker/nginx/nginx.conf /etc/nginx/nginx.conf
