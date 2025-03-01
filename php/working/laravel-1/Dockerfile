FROM php:8.2-fpm

ARG user
ARG uid

# Install system dependencies
RUN apt-get update && apt-get install -y \
    git \
    curl \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    zip \
    unzip \
    supervisor \
    nginx \
    build-essential \
    openssl \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libzip-dev \
    librdkafka-dev \
    libpq-dev \
    openssh-server \
    sqlite3 \
    nano \
    cron \
    libicu-dev \
    libmariadb-dev \
    unzip \
    zip \
    zlib1g-dev \
    libjpeg-dev

# Install Node.js
RUN curl -fsSL https://deb.nodesource.com/setup_current.x | bash - && \
    apt-get install -y nodejs

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Install Kafka
RUN git clone https://github.com/arnaud-lb/php-rdkafka.git && \
    cd php-rdkafka && \
    phpize && \
    ./configure && \
    make all -j$(nproc) && \
    make install && \
    docker-php-ext-enable rdkafka && \
    cd .. && \
    rm -rf /php-rdkafka

# Install PHP Extensions
RUN docker-php-ext-configure gd --enable-gd --with-freetype --with-jpeg && \
    docker-php-ext-install zip mbstring exif pcntl bcmath gd intl gettext pdo pdo_mysql sockets pdo_pgsql opcache

# Install Redis and enable it
RUN pecl install redis && docker-php-ext-enable redis

# Get latest Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Create system user
RUN useradd -G www-data,root -u ${uid:-1000} -d /home/${user:-www-data} ${user:-www-data} && \
    mkdir -p /home/${user:-www-data}/.composer && \
    chown -R ${user:-www-data}:${user:-www-data} /home/${user:-www-data}

# Set working directory
WORKDIR /var/www

# Copy necessary files
COPY src/index.php /var/www/index.php
COPY src/php.ini $PHP_INI_DIR/conf.d/
COPY src/opcache.ini $PHP_INI_DIR/conf.d/

# Copy entrypoint script
COPY src/entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/entrypoint.sh

ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]

# Change ownership
RUN chown -R ${uid:-1000}:${uid:-1000} /var/www /var/log /etc/supervisor/conf.d/ $PHP_INI_DIR/conf.d/ /tmp

# Copy Composer dependencies
COPY composer.json composer.lock ./
RUN composer install --no-dev --optimize-autoloader --no-scripts

COPY . .

# Run Supervisor
CMD ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisord.conf"]
