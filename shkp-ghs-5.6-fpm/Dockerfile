FROM 5.6-fpm-9056
MAINTAINER Jody WAN <jody.kpw@gmail.com>

# PHP Timezone
ENV TZ=UTC

# Installing the 'apt-utils' package gets rid of the 'debconf: delaying package configuration, since apt-utils is not installed'
# error message when installing any other package with the apt-get package manager.
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    apt-utils \
 && rm -rf /var/lib/apt/lists/*

# Common
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get -qq install \
    locales \
    php-pear
    # curl \
  	# wget \
    # zip

# Set locale
RUN sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen

# Set timezone
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN tar x --file=/usr/src/php.tar.xz --directory=/usr/src/  php-5.6.32/php.ini-production php-5.6.32/php.ini-development \
    && cp /usr/src/php-5.6.32/php.ini-production /usr/local/etc/php/php.ini

# Install PHP Core Extensions,
# zlib1g-dev: zlib is a library implementing the deflate compression method found in gzip and PKZIP.
# libxml2-dev: Development files for the GNOME XML library
# libbz2-dev: high-quality block-sorting file compressor library - development
# libpcre3, libpcre3-dev: PCRE Library
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get -qq install \
    libbz2-dev \
    zlib1g-dev \
    libxml2-dev \
    libpcre3 \
    libpcre3-dev

# Install more PHP extensions
RUN docker-php-source extract \
    && docker-php-ext-install mysqli \
    && docker-php-ext-install bz2 \
    && docker-php-ext-install gettext \
    && docker-php-ext-install pdo pdo_mysql \
    && docker-php-ext-install bcmath \
    && docker-php-ext-install soap \
    && docker-php-ext-install zip \
    && docker-php-ext-install exif \
    && docker-php-ext-install sockets \
    && docker-php-ext-install opcache \
    && docker-php-ext-enable opcache \
    && docker-php-source delete

# PHP-FPM image with iconv, mcrypt and gd extensions
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get -qq install \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libmcrypt-dev \
    libpng-dev \
    && docker-php-ext-install -j$(nproc) iconv mcrypt \
    && docker-php-ext-configure gd --enable-gd-native-ttf --with-freetype-dir=/usr/lib/ --with-jpeg-dir=/usr/lib/ --with-png-dir=/usr/lib/ \
    && docker-php-ext-install -j$(nproc) gd

# LDAP is the Lightweight Directory Access Protocol
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get -qq install \
    libldap2-dev \
    && docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu \
	  && docker-php-ext-install ldap

# Clean
RUN apt-get clean
RUN rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /var/cache/*

VOLUME /usr/local/etc/php

# Add exposes ports
EXPOSE 9056

# Add entrypoint
COPY docker-php-entrypoint /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-php-entrypoint
ENTRYPOINT ["docker-php-entrypoint"]
WORKDIR /var/www/html
CMD ["php-fpm"]
