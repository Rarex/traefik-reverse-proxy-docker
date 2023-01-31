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
          - "traefik.http.routers.${COMPOSE_PROJECT_NAME}-service-name.rule=Host(`${COMPOSE_PROJECT_NAME}.localhost`) || Host(`front.dev`)"
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

````
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
          
````

Add network to source container

````
    source-service-name
        networks:
          -proxy
          
    ...      
    networks:
        proxy:
            external: true
            name: proxy
````

Now `source-service-name` can access `destination-service-name` by `${COMPOSE_PROJECT_NAME}.localhost` url

#### Change `localhost` to another domain

Url like `*.localhost` automatically resolved by browser to `127.0.0.1`.

If you want to use urls like `front.dev`, `back.dev` instead of `front.localhost` and `back.localhost`, you will need to add records to your `hosts` file manually

````
    127.0.0.1 front.dev back.dev
````

