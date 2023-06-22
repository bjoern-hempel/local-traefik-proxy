# LocalTraefikProxy

A local Traefik proxy that simplifies access to local Docker development instances.

## Features

- Local development
- HTTP/HTTPS support
- Self-signed certificates

## Run Locally

Clone the project and change directory

```bash
git clone git clone https://github.com/bjoern-hempel/local-traefik-proxy.git && cd local-traefik-proxy
```

### Start local traefik proxy

#### Create external network

Must be executed only once:

```bash
docker network create traefik
```

```bash
docker network ls | grep traefik
```

#### Start traefik proxy

```bash
docker compose up -d
```

#### List docker container

```bash
docker container ls
```

#### Open traefik dashboard

* Open [http://localhost:8080/dashboard/#/](http://localhost:8080/dashboard/#/)
* Respectively [http://traefik.localhost/dashboard/#/](http://traefik.localhost/dashboard/#/) ;)

#### Shutdown local traefik proxy

```bash
docker compose up -d
```

## Start demo simple 1

```bash
cd demo/simple1 && docker compose up -d
```

* Open [https://simple1.localhost](https://simple1.localhost/)

```bash
docker compose down
```

## Start demo simple 2

```bash
cd demo/simple2 && docker compose up -d
```

* Open [https://simple2.localhost](https://simple2.localhost/)

```bash
docker compose down
```

## Start own container

### Simple example

This is a minimal example with nginx within your docker compose setup.

#### `.env`

Make changes to your .env file:

```dotenv
# Namespace to use for host name variables (hostname safe)
NAMESPACE_UNDERLINE=de_ixno_simple_1

# Namespace to use for host name variables (hostname safe) (development)
NAMESPACE_HOSTNAME_UNDERLINE=${NAMESPACE_UNDERLINE}_development

# The URL of this project
URL_LOCAL=simple1.localhost

# Traefik network name
NETWORK_NAME_TRAEFIK=traefik
```

#### `docker-compose.yml`

Add labels to your docker-compose.yml settings. **Tip**: Use the file `docker-compose.override.yml` to work with it locally
only and disable the settings for productive work (`docker-compose.prod.yml`).

```yaml
# Use docker compose version 3.8
version: '3.8'

# configure services
services:

  # Serve the project 1.
  application:
    image: arm64v8/nginx:latest
    ...
    labels:
      # enable traefik
      - "traefik.enable=true"
      # middleware
      - "traefik.http.middlewares.${NAMESPACE_HOSTNAME_UNDERLINE}_https.redirectscheme.scheme=https"
      # simple 1 project (http)
      - "traefik.http.routers.${NAMESPACE_HOSTNAME_UNDERLINE}_http.entrypoints=web"
      - "traefik.http.routers.${NAMESPACE_HOSTNAME_UNDERLINE}_http.rule=Host(`${URL_LOCAL}`)"
      - "traefik.http.routers.${NAMESPACE_HOSTNAME_UNDERLINE}_http.middlewares=${NAMESPACE_HOSTNAME_UNDERLINE}_https"
      # simple 1 project (https)
      - "traefik.http.routers.${NAMESPACE_HOSTNAME_UNDERLINE}_https.entrypoints=websecure"
      - "traefik.http.routers.${NAMESPACE_HOSTNAME_UNDERLINE}_https.rule=Host(`${URL_LOCAL}`)"
      - "traefik.http.routers.${NAMESPACE_HOSTNAME_UNDERLINE}_https.tls=true"
      # network
      - "traefik.docker.network=${NETWORK_NAME_TRAEFIK}"
    networks:
      - traefik
    ...
    
# configure networks
networks:
  traefik: # ${NETWORK_NAME_TRAEFIK}
    external: true
    name: "${NETWORK_NAME_TRAEFIK}"
```

### Real life example

This is a real life example.

#### `.env`

Make changes to your .env file:

```dotenv
# Namespace to use for host name variables (hostname safe)
NAMESPACE_UNDERLINE=de_ixno_real

# Namespace to use for host name variables (hostname safe) (development)
NAMESPACE_HOSTNAME_UNDERLINE=${NAMESPACE_UNDERLINE}_development

# The local URL of this project
URL_LOCAL=real.localhost

# Traefik network name (local)
NETWORK_NAME_TRAEFIK_PUBLIC_LOCAL=traefik

# https port
PORT_HTTPS=443

# Expose api https port (To bypass the Traefik proxy or if it is not installed)
PORT_HTTPS_API_EXPOSE=44443
```

#### `docker-compose.yml`

Add labels to your docker-compose.yml settings. **Tip**: Use the file `docker-compose.override.yml` to work with it locally
only and disable the settings for productive work (`docker-compose.prod.yml`).

```yaml
version: "3.8"

# Configures the services
services:

  # Nginx to serve the app.
  nginx:
    ...
    labels:
      # enable traefik
      - "traefik.enable=true"
      # middleware
      - "traefik.http.middlewares.${NAMESPACE_HOSTNAME_UNDERLINE}_https.redirectscheme.scheme=https"
      - "traefik.http.middlewares.${NAMESPACE_HOSTNAME_UNDERLINE}_frame.headers.customFrameOptionsValue=sameorigin"
      # services (load balancer)
      - "traefik.http.services.${NAMESPACE_HOSTNAME_UNDERLINE}_https_lb.loadbalancer.server.port=${PORT_HTTPS}"
      - "traefik.http.services.${NAMESPACE_HOSTNAME_UNDERLINE}_https_lb.loadbalancer.server.scheme=https"
      # http layer -> redirect https
      - "traefik.http.routers.${NAMESPACE_HOSTNAME_UNDERLINE}_http.entrypoints=web"
      - "traefik.http.routers.${NAMESPACE_HOSTNAME_UNDERLINE}_http.rule=Host(`www.${URL_LOCAL}`)"
      - "traefik.http.routers.${NAMESPACE_HOSTNAME_UNDERLINE}_http.middlewares=${NAMESPACE_HOSTNAME_UNDERLINE}_https"
      # https layer
      - "traefik.http.routers.${NAMESPACE_HOSTNAME_UNDERLINE}_https.entrypoints=websecure"
      - "traefik.http.routers.${NAMESPACE_HOSTNAME_UNDERLINE}_https.rule=Host(`www.${URL_LOCAL}`)"
      - "traefik.http.routers.${NAMESPACE_HOSTNAME_UNDERLINE}_https.middlewares=${NAMESPACE_HOSTNAME_UNDERLINE}_frame"
      - "traefik.http.routers.${NAMESPACE_HOSTNAME_UNDERLINE}_https.service=${NAMESPACE_HOSTNAME_UNDERLINE}_https_lb"
      - "traefik.http.routers.${NAMESPACE_HOSTNAME_UNDERLINE}_https.tls=true"
      # network
      - "traefik.docker.network=${NETWORK_NAME_TRAEFIK_PUBLIC_LOCAL}"
    ...
    ports:
      - "${PORT_HTTPS_API_EXPOSE}:${PORT_HTTPS}"
    networks:
      - network-internal
      - network-traefik
        
  other-service:
    ...
    networks:
      - network-internal
...

networks:
  network-internal:
    external: false
    name: "${NAMESPACE_HOSTNAME}.network.internal"
  network-traefik:
    external: true
    name: "${NETWORK_NAME_TRAEFIK_PUBLIC_LOCAL}"

```

## Create your own certificate

```bash
openssl req -x509 -nodes -days 3650 -newkey rsa:4096 -keyout ./tools/certs/cert.key -out ./tools/certs/cert.crt -subj "/C=DE/ST=Saxony/L=Dresden/O=Ixnode/OU=IT/CN=localhost"
```
