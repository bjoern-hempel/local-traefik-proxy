# LocalTraefikProxy

A local Traefik proxy that simplifies access to local Docker development instances.

## Features

- Local development
- HTTP/HTTPS support
- Self-signed certificates

## Run Locally

Clone the project and change directory

```bash
git clone https://github.com/bjoern-hempel/local-traefik-proxy.git && cd local-traefik-proxy
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

### Example 1

Minimal example.

#### `.env`

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

```bash
# Use docker compose version 3.8
version: '3.8'

# configure services
services:

  # Serve the project 1.
  application:
    image: arm64v8/nginx:latest
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

## Create your own certificate

```bash
openssl req -x509 -nodes -days 3650 -newkey rsa:4096 -keyout ./tools/certs/cert.key -out ./tools/certs/cert.crt -subj "/C=DE/ST=Saxony/L=Dresden/O=Ixnode/OU=IT/CN=localhost"
```
