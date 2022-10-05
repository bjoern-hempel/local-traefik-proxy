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

## Create your own certificate

```bash
openssl req -x509 -nodes -days 3650 -newkey rsa:4096 -keyout ./tools/certs/cert.key -out ./tools/certs/cert.crt -subj "/C=DE/ST=Saxony/L=Dresden/O=Ixnode/OU=IT/CN=localhost"
```
