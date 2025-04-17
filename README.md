# docker-webserver
Default apache webserver with configurable user+group



![Insight4U](https://img.shields.io/badge/Insight-4U-f08060)
![Github commits](https://i4ubadges.chau.bet/github/commits/TimChaubet-I4U/docker-webserver/main?cache=600)
![Github last-commit](https://i4ubadges.chau.bet/github/last-commit/TimChaubet-I4U/docker-webserver/main?cache=600)
![Github stars](https://i4ubadges.chau.bet/github/stars/TimChaubet-I4U/docker-webserver?cache=600)
[![Github issues](https://i4ubadges.chau.bet/github/issues/TimChaubet-I4U/docker-webserver?cache=600)](https://github.com/TimChaubet-I4U/docker-webserver/issues?q=is%3Aissue%20)
[![Github open issues](https://i4ubadges.chau.bet/github/open-issues/TimChaubet-I4U/docker-webserver?cache=600)](https://github.com/TimChaubet-I4U/docker-webserver/issues)
[![Docker Pulls](https://i4ubadges.chau.bet/docker/pulls/insight4upublic/webserver?icon=docker&label=pulls)](https://hub.docker.com/r/insight4upublic/webserver/)
[![Docker Stars](https://i4ubadges.chau.bet/docker/stars/insight4upublic/webserver?icon=docker&label=stars)](https://hub.docker.com/r/insight4upublic/webserver/)
[![Docker Image Size](https://i4ubadges.chau.bet/docker/size/insight4upublic/webserver?icon=docker&label=image%20size)](https://hub.docker.com/r/insight4upublic/webserver/)

Base webserver with 2 external volumes : /config & /www \
/config holds all apache2 & php8.2 config files \
/www is the entire webroot \

[Github](https://github.com/TimChaubet-I4U/docker-webserver) [Dockerhub](https://hub.docker.com/repository/docker/insight4upublic/webserver)

| Variable                 | Default           | Description                                            |
|--------------------------|-------------------|--------------------------------------------------------|
| `APACHE_DOCUMENT_ROOT`   | `/www`            | Apache’s DocumentRoot                                  |
| `WEB_USER`               | `www-data`        | Linux user Apache runs as                              |
| `WEB_GROUP`              | `www-data`        | Linux group Apache runs as                             |
| `WEB_UID`                | `33`              | UID to use (creates user if missing)                   |
| `WEB_GID`                | `33`              | GID to use (creates group if missing)                  |
| `TZ`                     | `Europe/Brussels` | Timezone for PHP (`date.timezone`)                     |


minimal:
```
docker create \
      -p 4567:80 \
      -v /some/host/folder/www:/www \
      -v /some/host/folder/config:/config \
      trueosiris/webserver
```

more options:
```
docker create \
      -p 4567:80 \
      -v /some/host/folder/www:/www \
      -v /some/host/folder/config:/config \
      -e APACHE_DOCUMENT_ROOT=/www \
      -e PGID=983 \
      -e PUID=983 \
      -e TZ=Europe/Brussels \
      -e HOST_HOSTNAME=$(hostname) \
      -e HOST_IP=$(ip addr show enp0s3 | grep "inet\b" | awk '{print $2}' | cut -d/ -f1) \
      --name webserver  \
      --restart=unless-stopped \
      -v "/var/run/docker.sock:/var/run/docker.sock" \
      trueosiris/webserver
```

docker compose

``` yaml
x-volume-localtime:
  &etclocaltime
  type: 'bind'
  source: /etc/localtime
  target: /etc/localtime
  read_only: true
x-volume-webdefault-webroot:
  &webdefaultwebroot
  type: 'bind'
  source: ./web/default/www
  target: /www
  bind:
    create_host_path: true  
x-volume-webdefault-config:
  &webdefaultconfig
  type: 'bind'
  source: ./web/default/config
  target: /config
  bind:
    create_host_path: true

services:
  webserver:
    image: insight4upublic/webserver
    environment:
      - APACHE_DOCUMENT_ROOT=/www 
      - PGID=1000 
      - PUID=1000 
      - TZ=Europe/Brussels 
      - HOST_HOSTNAME=$(hostname) 
      - HOST_IP=$(ip addr show enp0s3 | grep "inet\b" | awk '{print $2}' | cut -d/ -f1) 

    restart: unless-stopped
    network_mode: bridge     
    volumes: 
      - <<: *etclocaltime
      - <<: *webdefaultwebroot 
      - <<: *webdefaultconfig 
    ports:
      - 8031:80  
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:80"]
      interval: 60s
      timeout: 10s
      retries: 5
```