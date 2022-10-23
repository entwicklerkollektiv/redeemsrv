# start small
FROM alpine:latest

ARG http_proxy
ARG https_proxy
ARG no_proxy

ENV http_proxy=$http_proxy
ENV HTTP_PROXY=$http_proxy
ENV https_proxy=$https_proxy
ENV HTTPS_PROXY=$https_proxy
ENV no_proxy=$no_proxy

# nginx listens on port 80
EXPOSE 80

# Install nginx and git-backend
RUN \
  echo "http  Proxy: $http_proxy" && \
  echo "https Proxy: $https_proxy" && \
  echo "no    Proxy: $no_proxy" && \
  apk add --update nginx && \
  apk add --update git-daemon && \
  apk add --update fcgiwrap && \
  apk add --update spawn-fcgi && \
  # link nginx default access.log/error.log to stdout/stderr for docker
  ln -sf /dev/stdout /var/log/nginx/access.log && \
  ln -sf /dev/stderr /var/log/nginx/error.log && \
  # delete unnecessary cache
  rm -rf /var/cache/apk/*

# add nginx-conf
COPY nginx.conf /etc/nginx/nginx.conf
COPY templates.json /www/
COPY stacks /www/stacks/

RUN \
  mkdir -p /www /git && \
  cd /www && \
  # initialize git and repo
  git config --global init.defaultBranch "master" && \
  git config --global user.email "root@redeemsrv-templates.local" && \
  git config --global user.name "template repo user" && \
  git init && \
  git add . && \
  git commit -m 'initial' && \
  # convert to bare repo
  mv .git /git/templates.git && \
  cd /git/templates.git && \
  git config --bool core.bare true && \
  rm -rf /www/stacks

# start fcgi and nginx
CMD \
  spawn-fcgi -s /run/fcgi.sock /usr/bin/fcgiwrap && \
    nginx -g "daemon off;"

# reset env vars from build
ENV http_proxy=
ENV HTTP_PROXY=
ENV https_proxy=
ENV HTTPS_PROXY=
ENV no_proxy=
