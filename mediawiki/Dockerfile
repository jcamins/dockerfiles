from	ubuntu:14.04

# Configure apt
run	echo 'deb http://us.archive.ubuntu.com/ubuntu/ trusty universe' >> /etc/apt/sources.list
run	echo 'Acquire::http::Proxy "http://172.17.42.1:3142";' > /etc/apt/apt.conf.d/90-apt-cacher.conf
run	apt-get -y update

# Install prereqs
run	LC_ALL=C DEBIAN_FRONTEND=noninteractive apt-get install -y supervisor nginx-light php5-fpm php5-intl php5-cli curl php-apc php5-gd php5-intl php5-sqlite git

# Add system service config
add	./nginx.conf /etc/nginx/nginx.conf
add	./supervisord.conf /etc/supervisor/conf.d/supervisord.conf
add	./fpm.conf /etc/php5/fpm/php-fpm.conf
add	./fpm-pool-www.conf /etc/php5/fpm/pool.d/www.conf

# Install mediawiki
run	mkdir /src
add	http://releases.wikimedia.org/mediawiki/1.23/mediawiki-1.23.3.tar.gz /src/mediawiki.tgz
run	cd /src && tar zxf mediawiki.tgz
run	ln -snf /src/mediawiki-1.23.3 /src/mediawiki
run	chown -R www-data:www-data /src/mediawiki/

# Add data directory and symlinks and run installer
run	mkdir /data
run	ln -s /data/LocalSettings.php /src/mediawiki/LocalSettings.php
run	rm -rf /src/mediawiki/images
run	ln -s /data/images /src/mediawiki/images

# Run installer
run	cd /src/mediawiki && php maintenance/install.php --conf=/data --dbpath=/data --dbtype=sqlite --pass=smwadminpass smwtest smwadmin

# Install SMW and add it to MW
run	cd /src/mediawiki && curl -sS https://getcomposer.org/installer | php && php composer.phar require mediawiki/semantic-media-wiki "~2.0" && php maintenance/update.php && echo "enableSemantics( 'example.org' );" >> /data/LocalSettings.php

# Add startup script
add	./mediawiki-start /usr/bin/mediawiki-start

expose	80
cmd	["/usr/bin/mediawiki-start"]

# vim:ts=8:noet:
