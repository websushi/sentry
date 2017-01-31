## Sentry Dokku setup

### Create an app

```
dokku apps:create sentry
```

### Create Postgres database and link to app

```
dokku postgres:create sentry
dokku postgres:link sentry sentry
```

### Create Redis database and link to app

```
dokku redis:create sentry
dokku redis:link sentry sentry
```

### Create Memcached backend (optional)

```
dokku memcached:create sentry
dokku memcached:link sentry sentry
```

### Port config URLs to static environment variables

There are some very important environment variables to be set:

`SENTRY_FILESTORE_DIR`:

This is the location of the file storage inside the container.

First, create a persistent storage directory on the Dokku host machine
and mount it in the app container:

```
mkdir /var/lib/dokku/data/storage/sentry
chown -R 32767:32767 /var/lib/dokku/data/storage/sentry
dokku storage:mount sentry /var/lib/dokku/data/storage/sentry:/var/lib/sentry/files
```

Second, set the environment variable:

```
dokku config:set --no-restart sentry SENTRY_FILESTORE_DIR=/var/lib/sentry/files
```

`SENTRY_SECRET_KEY`:

This is the secret key Sentry uses for all kinds of important parts, so
this is has to be random.

First, generate a long, random key, e.g. with openssl:

```
openssl rand -hex 50
```

Second, set the environment variable:

```
dokku config:set --no-restart sentry SENTRY_SECRET_KEY=<RANDOM VALUE FROM ABOVE>
```

`DATABASE_URL`:

```
dokku config:set --no-restart sentry SENTRY_POSTGRES_HOST=dokku-postgres-sentry
dokku config:set --no-restart sentry SENTRY_POSTGRES_PORT=5432
dokku config:set --no-restart sentry SENTRY_DB_NAME=sentry
dokku config:set --no-restart sentry SENTRY_DB_USER=postgres
```

Please extract the password from `DATABASE_URL` and then run:

```
dokku config:set --no-restart sentry SENTRY_DB_PASSWORD=<EXTRACTED PASSWORD HERE>
```

`REDIS_URL`:

```
dokku config:set --no-restart sentry SENTRY_REDIS_HOST=dokku-redis-sentry
dokku config:set --no-restart sentry SENTRY_REDIS_PORT=6379
dokku config:set --no-restart sentry SENTRY_REDIS_DB=0
```

Please extract the password from `REDIS_URL` and then run:

```
dokku config:set --no-restart sentry SENTRY_REDIS_PASSWORD=<EXTRACTED PASSWORD HERE>
```

`MEMCACHED_URL`:

```
dokku config:set --no-restart sentry SENTRY_MEMCACHED_HOST=dokku-memcached-sentry
dokku config:set --no-restart sentry SENTRY_MEMCACHED_PORT=11211
```

### Other environment variables

If you want Sentry to optionally send mails, you need to set the following
environemnt variables.

```
dokku config:set --no-restart sentry SENTRY_SERVER_EMAIL='root@localhost'
dokku config:set --no-restart sentry SENTRY_EMAIL_HOST='smtp.example.com'
dokku config:set --no-restart sentry SENTRY_EMAIL_PORT=465
dokku config:set --no-restart sentry SENTRY_EMAIL_USER=sentry@acme.com
dokku config:set --no-restart sentry SENTRY_EMAIL_PASSWORD="somepassword"
dokku config:set --no-restart sentry SENTRY_EMAIL_USE_TLS=true
```
