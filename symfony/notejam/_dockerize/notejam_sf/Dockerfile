FROM php:7.1-cli as builder-php
# Update ini settings so that composer install works
RUN echo "memory_limit=256M" > $PHP_INI_DIR/conf.d/php-docker.ini

# install php dependencies
RUN apt-get update && apt-get install -y \
	zlib1g-dev \
	git \
	unzip \
	zip \
	build-essential \
	libssl-dev \
	zlib1g-dev \
	libpng-dev \
	libjpeg-dev \
	libfreetype6-dev 

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer --version=1.0.0 && composer --version

WORKDIR /app
COPY . /app/

RUN composer install --no-dev --no-interaction

#RUN git rev-parse HEAD > /app/version

FROM php:7.1-fpm
RUN echo "memory_limit=256M" > $PHP_INI_DIR/conf.d/php-docker.ini

RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -yq \
	mariadb-client

RUN apt-get update && \
	apt-get install -y \
	zlib1g-dev \
	libfontconfig1 \
	libxrender1 \
	libxext6 && \
	rm -rf /var/lib/apt/lists/*

# install and enable php extensions
RUN docker-php-ext-install pdo pdo_mysql

# Copy results from builder-php
COPY --from=builder-php /app/ /app/

# Setup working directory
WORKDIR /app

RUN php app/console doctrine:schema:update --force
#RUN php app/console server:run

## Update permissions - excluding /app/var
#RUN find /app -type d \( -path /app/var \) -prune -o -exec chmod ug-w {} \;
#
# Add our php user
RUN useradd -ms /bin/bash php

# Update ownership
RUN mkdir /app/var && chown -R php:php /app/var/

USER php

# expose ports
EXPOSE 8000
