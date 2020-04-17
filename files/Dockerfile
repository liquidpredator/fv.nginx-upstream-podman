FROM alpine:3.7
ENV NGINX_VERSION 1.14.0
RUN set -ex \
&& apk add --no-cache \
supervisor \
git \
krb5 \
krb5-dev \
ca-certificates \
libressl \
pcre \
zlib \
&& apk add --no-cache --virtual .build-deps \
build-base \
linux-headers \
libressl-dev \
pcre-dev \
wget \
curl \
zlib-dev \
&& cd /tmp \
&& wget http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz \
&& tar xzf nginx-${NGINX_VERSION}.tar.gz \
&& git clone https://github.com/yaoweibin/nginx_upstream_check_module.git nginx-${NGINX_VERSION}/nginx_upstream_check_module \
&& cd nginx-${NGINX_VERSION} \
&& patch -p1 < /tmp/nginx-${NGINX_VERSION}/nginx_upstream_check_module/check_${NGINX_VERSION}+.patch
RUN cd /tmp/nginx-${NGINX_VERSION} \
&& ./configure \
\
--prefix=/etc/nginx \
--sbin-path=/usr/sbin/nginx \
--conf-path=/etc/nginx/nginx.conf \
--error-log-path=/var/log/nginx/error.log \
--pid-path=/var/run/nginx.pid \
--lock-path=/var/run/nginx.lock \
--user=nginx \
--group=nginx \
--with-threads \
--with-file-aio \
--with-http_ssl_module \
--with-http_v2_module \
--with-http_realip_module \
--with-http_addition_module \
--with-http_sub_module \
--with-http_dav_module \
--with-http_flv_module \
--with-http_mp4_module \
--with-http_gunzip_module \
--with-http_gzip_static_module \
--with-http_auth_request_module \
--with-http_random_index_module \
--with-http_secure_link_module \
--with-http_slice_module \
--with-http_stub_status_module \
--http-log-path=/var/log/nginx/access.log \
--http-client-body-temp-path=/var/cache/nginx/client_temp \
--http-proxy-temp-path=/var/cache/nginx/proxy_temp \
--http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
--http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
--http-scgi-temp-path=/var/cache/nginx/scgi_temp \
--with-mail \
--with-mail_ssl_module \
--with-stream \
--with-stream_ssl_module \
--with-stream_realip_module \
--add-module=nginx_upstream_check_module \
&& make -j$(getconf _NPROCESSORS_ONLN) \
&& make install \
&& sed -i -e 's/#access_log logs\/access.log main;/access_log \/dev\/stdout;/' -e 's/#error_log logs\/error.log notice;/error_log stderr notice;/' /etc/nginx/nginx.conf \
&& adduser -D nginx \
&& mkdir -p /var/cache/nginx \
&& apk del .build-deps \
&& rm -rf /tmp/* \
&& mkdir /etc/nginx/cache \
&& mkdir /etc/nginx/certs \
&& mkdir /etc/nginx/vhosts \
&& mkdir /root/.acme.sh
ADD supervisord.conf /etc/
RUN echo '53 0 * * * "/root/.acme.sh"/acme.sh --cron --home "/root/.acme.sh" > /dev/null' > /etc/crontabs/root
ENTRYPOINT ["supervisord", "--nodaemon", "--configuration", "/etc/supervisord.conf"]
