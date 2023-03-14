# Konga

Kong Enterprise has a GUI for managing Services, Routes, and other configuration called Kong Manager. But because we’re using the community edition, we don’t have that feature.

Konga is basically an alternative GUI for interacting with Kong Admin API. The good thing is that it’s open source, unlike Kong Manager.

## Docker Compose

```yaml
version: "3.8"

services:
  konga:
    image: pantsel/konga
    environment:
      TOKEN_SECRET: ${KONGA_TOKEN_SECRET}
      DB_ADAPTER: ${KONG_DATABASE}
      DB_HOST: ${KONGA_DB_HOST}
      DB_PORT: ${KONGA_DB_PORT}
      DB_DATABASE: ${KONGA_DB_NAME}
      DB_USER: ${KONGA_DB_USERNAME}
      DB_PASSWORD: ${KONGA_DB_PASSWORD}
      NODE_ENV: ${KONGA_ENV}
      KONGA_HOOK_TIMEOUT: 10000
    restart: on-failure
    ports:
      - $KONGA_PORT:1337
    depends_on:
      - db
    networks:
      - kong-net
```

```env
KONGA_TOKEN_SECRET=some-secret-token
KONGA_ENV=development
KONGA_PORT=9000
```

## Setup Kong Loopback

Add Kong Admin API as services

```bash
curl --location --request POST 'http://localhost:8001/services/' \
--header 'Content-Type: application/json' \
--data-raw '{
    "name": "admin-api",
    "host": "localhost",
    "port": 8001
}'
```

Add Admin API route: To register route on Admin API Services we need either service name or service id, you can replace the following command below:

```bash
curl --location --request POST 'http://localhost:8001/services/{service_id|service_name}/routes' \
--header 'Content-Type: application/json' \
--data-raw '{
    "paths": ["/admin-api"]
}'
```

## Enable Key Auth Plugin

```bash
curl -X POST http://localhost:8001/services/{service_id|service_name}/plugins \
    --data "name=key-auth"
```

## Add Konga as the consumer

```bash
curl --location --request POST 'http://localhost:8001/consumers/' \
--form 'username=konga' \
--form 'custom_id=cebd360d-3de6-4f8f-81b2-31575fe9846a'
```

## Create API Key for Konga

```bash
curl --location --request POST 'http://localhost:8001/consumers/e7b420e2-f200-40d0-9d1a-a0df359da56e/key-auth'
```

# References

https://dev.to/vousmeevoyez/setup-kong-konga-part-2-dan
