# Deploying KONG

# Before getting started…

## Persistence Model

In the documentation, we will be working in DB Mode, and using PostgreSQL 13.

We can discuss DB-Less Mode in a later section.

## Kong Enterprise vs Kong Community Edition

It is important to note that there is also an enterprise version of Kong, that provides many more features and plugins, like Kong Manager, which is a browser-based GUI where you can configure and manage your Kong Gateway graphically.

However, for our course, we will only be using the **community edition**, also referred to as **Kong Gateway (OSS)**.

Enterprise docker image: `kong/kong-gateway:3.0.0.0-alpine`

Community docker image: `kong:3.0.0-alpine`

# Option 1: Deploying with Docker

You need to run three containers IN THE CORRECT ORDER.

First is the KONG database.

The run the KONG migration container to setup the database schema. Otherwise, running the next container will yield an error.

> Tip: Delete your docker volumes if you are having errors at this point. The volume might still be configured with an incompatible version or other configurations.

Then you can deploy the KONG container itself.

```bash
docker network create kong-net
```

```bash
docker run -d --name kong-database \
  --network=kong-net \
  -p 5432:5432 \
  -e "POSTGRES_USER=kong" \
  -e "POSTGRES_DB=kong" \
  -e "POSTGRES_PASSWORD=kongpass" \
  postgres:13
```

```bash
docker run --rm --network=kong-net \
  -e "KONG_DATABASE=postgres" \
  -e "KONG_PG_HOST=kong-database" \
  -e "KONG_PG_PASSWORD=kongpass" \
  -e "KONG_PASSWORD=test" \
 kong/kong-gateway:3.2.1.0 kong migrations bootstrap
```

```bash
docker run -d --name kong-gateway \
  --network=kong-net \
  -e "KONG_DATABASE=postgres" \
  -e "KONG_PG_HOST=kong-database" \
  -e "KONG_PG_USER=kong" \
  -e "KONG_PG_PASSWORD=kongpass" \
  -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
  -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
  -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
  -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
  -e "KONG_ADMIN_LISTEN=0.0.0.0:8001" \
  -e "KONG_ADMIN_GUI_URL=http://localhost:8002" \
  -e KONG_LICENSE_DATA \
  -p 8000:8000 \
  -p 8443:8443 \
  -p 8001:8001 \
  -p 8444:8444 \
  -p 8002:8002 \
  -p 8445:8445 \
  -p 8003:8003 \
  -p 8004:8004 \
  kong/kong-gateway:3.2.1.0
```

# Option 2: Deploying with Docker Compose

Similar to the method above, in a docker-compose file.

This might be a little confusing, so I’d recommend creating your own docker-compose based on the commands provided in Option 1.

# Exposing Your Services

For this demo, we have 3 containers running: kong-database, kong-gateway and our task-service Spring Boot API from our previous lesson.

| Key | Value |
| --- | --- |
| Kong Admin URL | http://localhost:8001 |
| Task Service Host | http://taskservice.com |

# Creating a Kong Service

To create a service that will bind (connect) to your upstream (API), execute the command below.

```bash
curl -i -X POST \
	—-url http://localhost:8001/services/ \
	—-data 'name=task-service' \
	—-data 'url=http://110.74.194.123:6072'
```

We are basically sending a POST request to Kong Gateway and telling it to create a `Service` that will point to our Spring Boot API at `[http://taskservice.com](http://taskservice.com)`

Note that [`http://localhost:8001/services`](http://localhost:8001/services) is our admin service endpoint.

# Creating a Kong Route

```bash
curl -i -X POST \
	—-url http://localhost:8001/services/task-service/routes \
	—-data 'name=task-api' \
	--data 'paths[]=/' \
	--data 'strip_path=false'
```

After creating a service, we can then create a route that will point to that service, by sending a POST request to the endpoint above.

`hosts[]=taskservice.com` is just a name that we set for the Route.

`paths[]=/` is the endpoint that we will create that will point to the service. It basically maps the routes as shown below, and we can use any other nested routes.

We can discuss `strip_path` in a later section.

http://localhost:8000/                             ->         http://110.74.194.123:6072/

http://localhost:8000/api/v1/tasks         ->         http://110.74.194.123:6072/api/v1/tasks

# Testing Our Configuration

If we send a request to the URL below, it should send the request to the API and send a response back to the client.

```bash
curl http://localhost:8000/api/v1/tasks \
	-H 'accept: application/json' \
	-H 'Content-Type: application/json' \
	-d '{
	  "title": "new group",
	  "description": "my new group for work",
	  "assignedTo": "5e97b8e3-3fad-457c-8c1b-dbc0d6a595e8",
	  "groupId": "c3de1e96-92b7-4a2e-a020-848f5d19d6fe"
	}'
```

You should get the response from the server.

```json
{
  "group": "c3de1e96-92b7-4a2e-a020-848f5d19d6fe"
}
```

# Strip Path

You may notice we’ve set `strip_path=false` when creating our route. What does it do?

By default, for every new route created, the default setting is for `strip_path` is `true`. This is useful if we want to configure a path that looks like this:

`http://localhost:8000/task-service/api/v1/tasks`

If that is the case, we have to run this command to create the route.

```bash
curl -i -X POST \
	—-url http://localhost:8001/services/task-service/routes \
	—-data 'name=task-service' \
	--data 'paths[]=/task-service'
```

Because we’ve set `paths[]=/task-service`, and `strip_path=true` by default, our route would look like this.

http://localhost:8000/***task-service***/api/v1/tasks      ->      http://110.74.194.123:6072/api/v1/tasks

You can conclude that `strip_path` cuts out or removes the path (`/task-service`) from the route and what is remaining is sent to the upstream server.
