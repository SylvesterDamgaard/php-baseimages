FROM php:8.1-apache

LABEL maintainer="Sylvester Nielsen"

WORKDIR /var/www/html

ENV APACHE_DOCUMENT_ROOT=/var/www/html/public
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=UTC

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone # buildkit

RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y libvips-dev exiftool ffmpeg ghostscript wget nano gnupg gosu curl ca-certificates zip unzip git sqlite3 libcap2-bin libpng-dev python2 libfreetype6-dev libjpeg62-turbo-dev libpng-dev libzip-dev && \
    php -r "readfile('http://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer && \
    curl -sL https://deb.nodesource.com/setup_12.x | bash - && \
    apt-get install -y nodejs && \
    curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - && \
    echo "deb https://dl.yarnpkg.com/debian/ stable main" > /etc/apt/sources.list.d/yarn.list && \
    apt-get update && apt-get install -y yarn && \
    apt-get -y autoremove && apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

#RUN pecl channel-update https://pecl.php.net/channel.xml && pecl install swoole
RUN pecl install -o -f redis && docker-php-ext-enable redis
#RUN wget https://github.com/libvips/libvips/releases/download/v8.11.3/vips-8.11.3.tar.gz && tar xzf vips-8.11.3.tar.gz && cd vips-8.11.3 && ./configure && make && make install && cd .. && rm -rf vips-8.11.3 vips-8.11.3.tar.gz
#RUN pecl install -o -f vips && docker-php-ext-enable vips

RUN docker-php-ext-configure gd --with-freetype --with-jpeg && \
    docker-php-ext-install -j$(nproc) gd

RUN docker-php-ext-install exif && \
    docker-php-ext-install sockets && \
    docker-php-ext-install bcmath && \
    docker-php-ext-install zip && \
    docker-php-ext-install pcntl && \
    docker-php-ext-configure opcache --enable-opcache && \
    docker-php-ext-install opcache && \
    docker-php-ext-install mysqli pdo pdo_mysql

RUN a2enmod rewrite headers
RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf
RUN sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf
RUN mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"
#COPY php.ini /usr/local/etc/php/php-bazo.ini
#RUN cat /usr/local/etc/php/php-bazo.ini >> /usr/local/etc/php/php.ini

RUN sed -i 's/memory_limit = .*/memory_limit = 512M/' /usr/local/etc/php/php.ini
RUN sed -i 's/post_max_size = .*/post_max_size = 100M/' /usr/local/etc/php/php.ini
RUN sed -i 's/upload_max_filesize = .*/upload_max_filesize = 100M/' /usr/local/etc/php/php.ini
RUN sed -i 's/variables_order = .*/variables_order = EGPCS/' /usr/local/etc/php/php.ini
RUN sed -i 's/max_execution_time = .*/max_execution_time = 70/' /usr/local/etc/php/php.ini