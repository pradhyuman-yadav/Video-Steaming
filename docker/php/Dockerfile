# Use an official PHP FPM image as a parent image (adjust PHP version if needed)
FROM php:8.1-fpm

# Set working directory
WORKDIR /var/www/html

# Install system dependencies
RUN apt-get update && apt-get install -y \
    git \
    curl \
    libpng-dev \
    libjpeg62-turbo-dev \
    libfreetype6-dev \
    libonig-dev \
    libxml2-dev \
    libzip-dev \
    zip \
    unzip \
    jpegoptim optipng pngquant gifsicle \
    vim \
    nano \
    # For Intervention Image (gd or imagick)
    libmagickwand-dev --no-install-recommends \
    && pecl install imagick \
    && docker-php-ext-enable imagick

# Adjust source path if needed
COPY docker/php/conf.d/uploads.ini /usr/local/etc/php/conf.d/uploads.ini 


# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Install PHP extensions based on composer.json requirements
RUN docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) gd \
    && docker-php-ext-install pdo pdo_mysql mbstring exif pcntl bcmath zip opcache mysqli 

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Add user for laravel application
RUN groupadd -g 1000 www && \
    useradd -u 1000 -ms /bin/bash -g www www

# Copy existing application directory contents
COPY . /var/www/html

# After copying code:
# Set ownership to www-data (adjust user/group if different in your base image)
# Set permissions (775 allows user/group write, 755 allows only user write)
RUN chown -R www-data:www-data /var/www/html/storage /var/www/html/bootstrap/cache /var/www/html/public /var/www/html/resources && \
    chmod -R 775 /var/www/html/storage /var/www/html/bootstrap/cache /var/www/html/public /var/www/html/resources

# Copy existing application directory permissions
COPY --chown=www:www . /var/www/html

# Change current user to www
USER www

# Install composer dependencies
# --no-dev is recommended for production builds
# --optimize-autoloader and --no-interaction are good practices
RUN composer install --optimize-autoloader --no-dev --no-interaction

# Switch back to root to change permissions if necessary
USER root

# Adjust permissions for Laravel
RUN mkdir -p /var/www/html/storage/framework/sessions \
             /var/www/html/storage/framework/views \
             /var/www/html/storage/framework/cache \
             /var/www/html/storage/logs \
             /var/www/html/bootstrap/cache \
             /var/www/html/resources \
             /var/www/html/public && \
    chown -R www-data:www-data /var/www/html/storage /var/www/html/bootstrap/cache /var/www/html/resources /var/www/html/public && \
    chmod -R 775 /var/www/html/storage /var/www/html/bootstrap/cache /var/www/html/resources /var/www/html/public

# Expose port 9000 and start php-fpm server
EXPOSE 9000
CMD ["php-fpm"]