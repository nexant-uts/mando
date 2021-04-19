FROM ubuntu:20.04

LABEL maintainer="John Hoopes"

ARG PHP=7.4

RUN mkdir /app
RUN useradd -ms /bin/bash -u 1337 developer
WORKDIR /app

# Avoid prompts while building
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=UTC
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt-get update \
    && apt-get install -y gosu gnupg ca-certificates wget curl gzip zip unzip git \
    supervisor sqlite3 fontconfig libxrender1 xfonts-75dpi xfonts-base \
    && echo "ttf-mscorefonts-installer msttcorefonts/accepted-mscorefonts-eula select true" | debconf-set-selections \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y ttf-mscorefonts-installer \
    && fc-cache -f \
    && wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6-1/wkhtmltox_0.12.6-1.focal_amd64.deb \
    &&  apt install -y ./wkhtmltox_0.12.6-1.focal_amd64.deb \
    && rm ./wkhtmltox_0.12.6-1.focal_amd64.deb


RUN apt-get update \
    && echo "deb http://ppa.launchpad.net/ondrej/php/ubuntu focal main" > /etc/apt/sources.list.d/ppa_ondrej_php.list \
    && apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys E5267A6C \
    && apt-get update \
    && apt-get install -y nginx php$PHP-fpm php$PHP-cli \
        php$PHP-pgsql php$PHP-sqlite3 php$PHP-gd \
        php$PHP-curl php$PHP-memcached \
        php$PHP-imap php$PHP-mysql php$PHP-mbstring \
        php$PHP-xml php$PHP-zip php$PHP-bcmath php$PHP-soap \
        php$PHP-intl php$PHP-readline php$PHP-xdebug php$PHP-pcov \
        php$PHP-msgpack php$PHP-igbinary  php$PHP-imagick \
        php$PHP-ldap php$PHP-gmp \
        php$PHP-redis \
    && php -r "readfile('http://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer \
    && mkdir -p /run/php \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
    && sed -i "s/pm\.max_children = .*/pm.max_children = 20/" /etc/php/$PHP/fpm/pool.d/www.conf \
    && sed -i "s/pm\.start_servers = .*/pm.start_servers = 10/" /etc/php/$PHP/fpm/pool.d/www.conf \
    && sed -i "s/pm\.min_spare_servers = .*/pm.min_spare_servers = 5/" /etc/php/$PHP/fpm/pool.d/www.conf \
    && sed -i "s/pm\.max_spare_servers = .*/pm.max_spare_servers = 10/" /etc/php/$PHP/fpm/pool.d/www.conf \
    && echo "daemon off;" >> /etc/nginx/nginx.conf \
    && ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log \
    && sed -i 's/^;daemonize.*$/daemonize = no/g' /etc/php/$PHP/fpm/php-fpm.conf \
    && sed -i 's@^error_log.*$@error_log = /proc/self/fd/2@g' /etc/php/$PHP/fpm/php-fpm.conf

COPY default /etc/nginx/sites-available/default
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY xdebug.ini /etc/php/$PHP/fpm/conf.d/99-custom_xdebug.ini
COPY xdebug.ini /etc/php/$PHP/cli/conf.d/99-custom_xdebug.ini

COPY entrypoint /usr/local/bin/entrypoint
RUN chmod +x /usr/local/bin/entrypoint


EXPOSE 80

ENTRYPOINT ["entrypoint"]
