# Traefik reverse proxy docker configuration for development

## Usage

### Traefik setup

#### Creeate .env file from .env.example (optional)

```sh
cp .env.example .env
```

---

### Your projects setup

#### Disable port mapping used by traefik

```
#    ports:
#      - "80:80"
#      - "443:443"
#      - "8080:8080"
```

#### Add labels to your services

`${COMPOSE_PROJECT_NAME}` - By default: the basename of the project directory

```
    service-name
        ...
        labels:
          - "traefik.enable=true"
          - "traefik.http.routers.${COMPOSE_PROJECT_NAME}-service-name.rule=Host(`${COMPOSE_PROJECT_NAME}.localhost`)"
```

Change service port if needed

```
          - "traefik.http.services.${COMPOSE_PROJECT_NAME}-service-name.loadbalancer.server.port=5173"
```
Multiple hosts rule example

```
    service-name
        ...
        labels:
          - "traefik.enable=true"
          - "traefik.http.routers.${COMPOSE_PROJECT_NAME}-service-name.rule=Host(`${COMPOSE_PROJECT_NAME}.localhost`) || Host(`front.dev.local`)"
```


#### Add network

Traefik network name can be changed in `.env` file

```
    service-name
        networks:
          - proxy
    ...      
    networks:
        proxy:
            external: true
            name: proxy
```

#### If you need interaction between services from different containers

Add network and alias to destination service

```
    destination-service-name
        ...
        networks:
          proxy:
            aliases:
              - ${COMPOSE_PROJECT_NAME}.localhost
    ...      
    networks:
        proxy:
            external: true
            name: proxy
          
```

Add network to source container

```
    source-service-name
        networks:
          -proxy
          
    ...      
    networks:
        proxy:
            external: true
            name: proxy
```

Now `source-service-name` can access `destination-service-name` by `${COMPOSE_PROJECT_NAME}.localhost` url

#### Change `localhost` to another domain

Url like `*.localhost` automatically resolved by browser to `127.0.0.1`.

If you want to use urls like `front.dev.local`, `back.dev.local` instead of `front.localhost` and `back.localhost`, you will need to add records to your `hosts` file manually

```
    127.0.0.1 front.dev.local back.dev.local
```

Or use some DNS Proxy (e.g `Acrylic DNS Proxy`), so u can use wildcards config

```
    127.0.0.1 *.dev.local
```

Using `Acrylic` with `WSL2` may cause some issues with port `:53` used by them. Change `Acrylic` config `LocalIPv4BindingAddress=0.0.0.0` to `LocalIPv4BindingAddress=127.0.0.1` to solve it.

#### Using https
Http to https redirect enabled for all hosts by default. If you want to control redirect manually comment following lines at `docker-compose.yml`
```
#      - --entrypoints.web.http.redirections.entryPoint.to=websecure
#      - --entrypoints.web.http.redirections.entryPoint.scheme=https
#      - --entrypoints.web.http.redirections.entryPoint.permanent=true
```
Add labels to service
```
    service-name
        ...
        labels:
          ...
          - "traefik.http.middlewares.redirect-to-https.redirectscheme.scheme=https"
          - "traefik.http.middlewares.redirect-to-https.redirectscheme.permanent=true"
          - "traefik.http.routers.${COMPOSE_PROJECT_NAME}-service-name.middlewares=redirect-to-https"
```

#### Https self-signed certificates for local development
1. Generate root certificates in `./certs` directory
```sh
./cert_root.sh
```
2. Create domains.ext file
```sh
cp domains.ext.example domains.ext
```
3. Edit `domains.ext`. Change `DNS.1 = *.dev.local` to your host if needed. (I had problems with `Chrome` and first level wildcard certificates like `*.localhost` or `*.local`, so I changed it to `*.dev.local`)
4. Generate certificates for specified domains 
```sh
./cert.sh
```
5. Install root certificate in your operating system or browser (e.g. `Mozilla Firefox` uses its own certificate storage, when `Chrome` uses  certificates installed in operating system) 