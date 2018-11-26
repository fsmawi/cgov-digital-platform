FROM ubuntu:bionic

################################################################################
#                                                                              #
#   THIS CONTAINER IS FOR EDUCATIONAL USE ONLY!                                #
#                                                                              #
#   Never use this container for running production workloads!                 #
#                                                                              #
################################################################################

# Update the package manager...
RUN DEBIAN_FRONTEND=noninteractive \
    apt-get update

# ...then install packages.
RUN DEBIAN_FRONTEND=noninteractive \
    apt-get install -yq \
    apache2 \
    php7.2 \
    libapache2-mod-php7.2 \
    php7.2-bz2 \
    php7.2-cli \
    php7.2-common \
    php7.2-curl \
    php7.2-fpm \
    php7.2-gd \
    php7.2-json \
    php7.2-mbstring \
    php7.2-memcached \
    php7.2-mysql \
    php7.2-oauth \
    php7.2-opcache \
    php7.2-readline \
    php7.2-sqlite3 \
    php7.2-soap \
    php7.2-xdebug \
    php7.2-xml \
    mariadb-client \
    curl \
    git \
    vim \
    zip

# Set Timezone
RUN ln -fs /usr/share/zoneinfo/America/New_York /etc/localtime && \
  dpkg-reconfigure --frontend noninteractive tzdata


# Replace configuration files with our own.
## We will be mounting these in the composer so that
## we don't have to rebuild to update config changes.
#COPY apache2.conf /etc/apache2/apache2.conf
#COPY php_apache2.ini /etc/php/7.2/apache2/php.ini
#COPY php_cli.ini /etc/php/7.2/cli/php.ini

# Copy up the available sites config. The startup script
# manipulates this file based on env vars.
COPY ./build/000-default.conf /etc/apache2/sites-available/000-default.conf

## These are the PHP modules that are available to be
## loaded by the CLI & apache. This should be inline with
## the modules installed above.
COPY ./build/php-conf /etc/php/7.2/mods-available

# Enable mod_rewrite 'cause we needs it.
RUN a2enmod rewrite

# Forward logs to docker log collector.
RUN ln -sf /dev/stdout /var/log/apache2/access.log && \
    ln -sf /dev/stderr /var/log/apache2/error.log && \
    ln -sf /dev/stdout /var/log/apache2/000_default-access_log && \
    ln -sf /dev/stderr /var/log/apache2/000_default-error_log

# Copy our custom entrypoint and make it executable.
COPY ./build/docker-entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-entrypoint.sh

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Make composer faster
RUN composer global require "hirak/prestissimo:^0.3"

## Install Drush Launcher (runs local drush instances)
RUN curl -sL -o /usr/local/bin/drush https://github.com/drush-ops/drush-launcher/releases/download/0.6.0/drush.phar \
    && chmod +x /usr/local/bin/drush

## Install Drupal console
RUN curl https://drupalconsole.com/installer -L -o /usr/local/bin/drupal \
    && chmod +x /usr/local/bin/drupal

RUN php --version; composer --version; drupal --version; drush --drush-launcher-version

## Enable BLT function
COPY ./build/blt.sh /tmp/blt.sh
RUN cat /tmp/blt.sh >> ~/.bashrc

## Turn off xdebug as it uses resources that could be best used
## elsewhere. Once the container is up you can use
## phpenmod -s <SAPI> xdebug
## where SAPI is probably, most likely, apache2
RUN phpdismod -s cli xdebug
RUN phpdismod -s apache2 xdebug

# Expose the default Apache port.
EXPOSE 80

# Replace the standard entrypoint /bin/sh with our script.
ENTRYPOINT ["docker-entrypoint.sh"]

# If no command is passed to the container, start Apache by default.
CMD ["apachectl", "-D", "FOREGROUND"]