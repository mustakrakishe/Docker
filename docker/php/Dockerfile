FROM php:7.4-fpm

COPY --from=composer:2.3.5 /usr/bin/composer /usr/bin/composer

WORKDIR /var/www/html

RUN pecl install xdebug-3.1.6

RUN apt-get update && apt-get install libaio1 unzip \
    && mkdir /opt/oracle \
    && curl https://download.oracle.com/otn_software/linux/instantclient/instantclient-basic-linuxx64.zip \
        --output /opt/oracle/instantclient-sdk-linuxx64.zip \
    && unzip /opt/oracle/instantclient-sdk-linuxx64.zip -d /opt/oracle \
    && rm /opt/oracle/instantclient-sdk-linuxx64.zip \
    && curl https://download.oracle.com/otn_software/linux/instantclient/instantclient-sdk-linuxx64.zip \
        --output /opt/oracle/instantclient-sdk-linuxx64.zip \
    && unzip /opt/oracle/instantclient-sdk-linuxx64.zip -d /opt/oracle \
    && rm /opt/oracle/instantclient-sdk-linuxx64.zip \
    && export LD_LIBRARY_PATH=$(find /opt/oracle -mindepth 1 -maxdepth 1) \
    && echo ${LD_LIBRARY_PATH} > /etc/ld.so.conf.d/oracle-instantclient.conf \
    && ldconfig \
    && printf "instantclient,"${LD_LIBRARY_PATH} | pecl install oci8-2.2.0 \
    && docker-php-ext-enable oci8

# Imagick
RUN apt-get -y install libmagickwand-dev --no-install-recommends \
    && printf "\n" | pecl install imagick \
    && docker-php-ext-enable imagick

# Composer required packages
RUN apt-get install -y libzip-dev \
     && docker-php-ext-install zip gd

# MySQL
RUN docker-php-ext-install pdo_mysql

# Cron
RUN apt-get install -y cron \
    && echo "* * * * * /usr/local/bin/php /var/www/html/artisan schedule:run >> /var/www/html/storage/logs/cron.log 2>&1" > /etc/cron.d/run_laravel_scheduler \
    && crontab -u www-data /etc/cron.d/run_laravel_scheduler

EXPOSE 9000

CMD cron && php-fpm