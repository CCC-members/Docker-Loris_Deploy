FROM php:7.1-apache-jessie
LABEL maintainer="mario462@gmail.com"

#Install necessary software
RUN apt-get update && \
    apt-get install -yy --no-install-recommends sudo zip curl wget software-properties-common libxml2-dev libpng-dev git unzip && \
    rm -rf /var/lib/apt/lists/*
#Install default php extensions
RUN docker-php-ext-install pdo pdo_mysql mysqli xml json mbstring gd
#Configure and install non default php ast extension
RUN git clone https://github.com/nikic/php-ast /usr/src/php/ext/ast/ && \
    docker-php-ext-configure ast && \
    docker-php-ext-install ast
#Install composer
RUN curl -sS https://getcomposer.org/installer | php
RUN mv composer.phar /usr/local/bin/composer
#Add sudo and sudo user lorisadmin
RUN adduser --disabled-password --gecos '' lorisadmin
RUN adduser lorisadmin sudo
RUN echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
#Download loris project into /var/www directory
WORKDIR /var/www/
ARG lorisVersion=20.2.0
RUN wget https://github.com/aces/Loris/archive/v${lorisVersion}.zip -O release.zip && \
    unzip release.zip && \
    rm release.zip && \
    mv Loris* loris
#Replace install script with custom one that supports Debian OS
ADD install.sh loris/tools/install.sh
#Change owner to loris directory and set execution permission for custom install script
RUN chown -R lorisadmin.lorisadmin loris && \
    chmod +x loris/tools/install.sh
WORKDIR /var/www/loris

