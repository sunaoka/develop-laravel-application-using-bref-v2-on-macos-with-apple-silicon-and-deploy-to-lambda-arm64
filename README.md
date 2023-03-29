# Develop Laravel application (using bref v2) on macOS with Apple silicon and deploy to Lambda (arm64).

## Local development with docker

### Create a new Laravel project

```bash
composer create-project laravel/laravel develop-laravel-application-using-bref-v2-on-macos-with-apple-silicon-and-deploy-to-lambda-arm64
```

### Add Bref and Bref Laravel Bridge

```bash
composer require bref/bref bref/laravel-bridge --update-with-dependencies
```

### Create a docker-compose.yaml

```yaml
services:
  app:
    image: bref/arm-php-82-fpm-dev:2
    user: nobody  # Unix user of FPM processes.
    ports:
      - '8000:8000'
    volumes:
      - .:/var/task:ro
    environment:
      HANDLER: public/index.php
```

> **Warning**
> You must always specify `nobody` user in docker-compose.yaml.
>
> Otherwise, the following error will occur:
>
> `file_put_contents(/tmp/storage/framework/views/a0720c85aa2eaf0d3af39f30002c583b.php): Failed to open stream: Permission denied`
>
> The reason is that `nobody` is specified as the FPM user in [php-fpm.conf](https://github.com/brefphp/aws-lambda-layers/blob/main/layers/fpm/php-fpm.conf).

#### Add Xdebug

Edit the `php/conf.dev.d/php.ini` file:

```ini
[xdebug]
zend_extension = xdebug.so
xdebug.mode = debug
xdebug.start_with_request = yes
xdebug.client_host = 'host.docker.internal'
```

### Create and start docker container

```bash
docker compose up
```

Once you have started docker container, your application will be accessible in your web browser at `http://localhost:8000`.

## Deployment

### Create a serverless.yml

```bash
php artisan vendor:publish --tag=serverless-config
```

#### Use arm64 architecture

```yaml
# serverless.yml
provider:
  ...
  # Use arm64 architecture (AWS Graviton2 processor)
  architecture: arm64
```

### Deploying

Remove the configuration cache file:

```bash
php artisan config:clear
```

> **Note**
> At booting in AWS Lambda, Bref Laravel Bridge automatically creates a cache file of the configuration.

Remove development dependencies and optimize Composer's autoloader for production:

```bash
composer install --prefer-dist --optimize-autoloader --no-dev
```

Let's deploy now:

```bash
serverless deploy --verbose
```
