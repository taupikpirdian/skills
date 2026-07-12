---
name: laravel-docker-server-deploy
description: Gunakan skill ini saat menjalankan atau menyusun deploy aplikasi Laravel ke server menggunakan Docker atau Docker Compose, dengan fokus pada build image dan menjalankan container release.
---

# Laravel server deploy with Docker

Skill ini fokus pada deploy aplikasi Laravel ke server menggunakan Docker.
Gunakan saat perlu:

- menjalankan deploy aplikasi ke server
- menyusun flow deploy berbasis Docker
- membuat atau merapikan script deploy

## Tujuan deploy

Deploy yang benar harus memastikan:

- image aplikasi terbaru berhasil di-build
- container yang berjalan memakai image terbaru
- dependency PHP dan asset frontend sudah ikut di image
- aplikasi bisa diakses normal setelah release

## Pola deploy yang disarankan

Untuk aplikasi Laravel berbasis Docker, urutan deploy yang aman adalah:

1. Pull source code terbaru di server.
2. Pastikan file environment production sudah benar.
3. Build image aplikasi terbaru.
4. Recreate container dengan image baru.
5. Pastikan service berjalan dengan image release terbaru.

## Contoh command deploy

Contoh command dasar:

```bash
docker compose build
docker compose up -d --remove-orphans
```

Jika container aplikasi harus dipastikan rebuild:

```bash
docker compose up -d --build --remove-orphans
```

## Contoh `deploy.sh`

Contoh script deploy yang bisa dipakai sebagai baseline:

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "[1/7] Build latest image"
docker compose build

echo "[2/3] Recreate containers"
docker compose up -d --build --remove-orphans

echo "[3/3] Show running services"
docker compose ps

echo "Deploy finished"
```

Catatan:

- gunakan `set -euo pipefail` agar script berhenti saat ada command gagal

## Contoh `docker-compose.yml`

Contoh minimal pola service Laravel di server:

```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: laravel-app
    restart: unless-stopped
    working_dir: /var/www
    volumes:
      - app_storage:/var/www/storage
      - app_cache:/var/www/bootstrap/cache
    networks:
      - app-network

  nginx:
    image: nginx:alpine
    container_name: laravel-nginx
    restart: unless-stopped
    ports:
      - "80:80"
    volumes:
      - ./public:/var/www/public
      - ./docker/nginx.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - app
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  app_storage:
  app_cache:
```

## Contoh `Dockerfile`

Contoh image Laravel berbasis PHP-FPM:

```dockerfile
FROM php:8.3-fpm

WORKDIR /var/www

RUN apt-get update && apt-get install -y \
    git \
    curl \
    unzip \
    zip \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    libzip-dev \
    libicu-dev \
    nodejs \
    npm \
    && docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd zip intl \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

COPY composer.json composer.lock package.json ./

RUN composer install \
    --no-interaction \
    --no-dev \
    --optimize-autoloader

RUN npm install

COPY . .

RUN npm run build

RUN chown -R www-data:www-data /var/www \
    && chmod -R 775 storage bootstrap/cache

CMD ["php-fpm"]
```

## Aturan penting saat deploy ke server

Saat deploy Laravel dengan Docker, ikuti aturan ini:

- anggap image sebagai artifact release
- jangan edit source code langsung di container production
- jangan mengandalkan perubahan lokal tanpa rebuild image
- pastikan konfigurasi port dan service sesuai kebutuhan server

## Mindset rollback

Jika belum ada automation rollback:

- rollback tercepat biasanya kembali ke image sebelumnya
- rollback database belum tentu aman
- migration harus backward compatible jika ingin rollback lebih aman
- jangan mengklaim rollback aman tanpa memeriksa schema change

## Ringkasan singkat

Deploy Laravel ke server dengan Docker berarti:

- build image terbaru
- recreate container
- jalankan service release terbaru

Fokus skill ini adalah release aplikasi Laravel ke server secara konsisten dan aman menggunakan Docker.
