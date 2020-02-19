FROM alpine:latest as stage-1

#Set mirrors
RUN echo "https://mirror.leaseweb.com/alpine/latest-stable/main" > /etc/apk/repositories \
	&& echo "https://mirror.leaseweb.com/alpine/latest-stable/community" >>/etc/apk/repositories \
	&& apk add --no-cache nginx supervisor php7-fpm php7-mcrypt php7-openssl \
        	php7-json php7-dom php7-zip php7-mysqli php7-ctype php7-intl \
       	 	php7-gd php7-xmlreader php7-xml php7-zlib  php7-phar \
	        php7-exif php7-fileinfo php7-mbstring php7-sodium \
	        php7-imagick php7-gd php7-simplexml \
        	php7-ftp php7-sockets \	
	&& mkdir -p /run/nginx \
	&& adduser -D -g 'www' www \
	&& mkdir /www \
	&& chown -R www:www /var/lib/nginx \
	&& chown -R www:www /www \
	&& mv /etc/nginx/nginx.conf /etc/nginx/nginx.conf.orig \
	&& rm -rf /var/cache/apk/*

#Define PHP environment
ENV PHP_FPM_USER="www" \
	PHP_FPM_GROUP="www" \
	PHP_FPM_LISTEN_MODE="0660" \
	PHP_MEMORY_LIMIT="512M" \
	PHP_MAX_UPLOAD="128M" \
	PHP_MAX_FILE_UPLOAD="200" \
	PHP_MAX_POST="128M" \
	PHP_DISPLAY_ERRORS="On" \
	PHP_DISPLAY_STARTUP_ERRORS="On" \
	PHP_ERROR_REPORTING="E_COMPILE_ERROR\|E_RECOVERABLE_ERROR\|E_ERROR\|E_CORE_ERROR" \
	PHP_CGI_FIX_PATHINFO=0

#Configure PHP
RUN sed -i "s|;listen.owner\s*=\s*nobody|listen.owner = ${PHP_FPM_USER}|g" /etc/php7/php-fpm.d/www.conf \
	&& sed -i "s|;listen.group\s*=\s*nobody|listen.group = ${PHP_FPM_GROUP}|g" /etc/php7/php-fpm.d/www.conf \
	&& sed -i "s|;listen.mode\s*=\s*0660|listen.mode = ${PHP_FPM_LISTEN_MODE}|g" /etc/php7/php-fpm.d/www.conf \
	&& sed -i "s|user\s*=\s*nobody|user = ${PHP_FPM_USER}|g" /etc/php7/php-fpm.d/www.conf \
	&& sed -i "s|group\s*=\s*nobody|group = ${PHP_FPM_GROUP}|g" /etc/php7/php-fpm.d/www.conf \
	&& sed -i "s|;log_level\s*=\s*notice|log_level = notice|g" /etc/php7/php-fpm.d/www.conf \
	&& sed -i "s|display_errors\s*=\s*Off|display_errors = ${PHP_DISPLAY_ERRORS}|i" /etc/php7/php.ini \
	&& sed -i "s|display_startup_errors\s*=\s*Off|display_startup_errors = ${PHP_DISPLAY_STARTUP_ERRORS}|i" /etc/php7/php.ini \
	&& sed -i "s|error_reporting\s*=\s*E_ALL & ~E_DEPRECATED & ~E_STRICT|error_reporting = ${PHP_ERROR_REPORTING}|i" /etc/php7/php.ini \
	&& sed -i "s|;*memory_limit =.*|memory_limit = ${PHP_MEMORY_LIMIT}|i" /etc/php7/php.ini \
	&& sed -i "s|;*upload_max_filesize =.*|upload_max_filesize = ${PHP_MAX_UPLOAD}|i" /etc/php7/php.ini \
	&& sed -i "s|;*max_file_uploads =.*|max_file_uploads = ${PHP_MAX_FILE_UPLOAD}|i" /etc/php7/php.ini \
	&& sed -i "s|;*post_max_size =.*|post_max_size = ${PHP_MAX_POST}|i" /etc/php7/php.ini \
	&& sed -i "s|;*cgi.fix_pathinfo=.*|cgi.fix_pathinfo= ${PHP_CGI_FIX_PATHINFO}|i" /etc/php7/php.ini

FROM stage-1 as final
COPY config/nginx.conf /etc/nginx/nginx.conf
COPY config/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY html/index.php /www/index.php

# Make sure files/folders needed by the processes are accessable when they run under the nobody user
RUN chown -R nobody.nobody /www && \
  chown -R nobody.nobody /run && \
  chown -R nobody.nobody /var/lib/nginx && \
  chown -R nobody.nobody /var/log/nginx

EXPOSE 80

STOPSIGNAL SIGTERM

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]

