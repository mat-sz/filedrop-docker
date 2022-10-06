# filedrop-docker

A docker-compose setup for filedrop. Contains: [filedrop-web](https://github.com/mat-sz/filedrop-web), [filedrop-ws](https://github.com/mat-sz/filedrop-ws), [coturn](https://github.com/coturn/coturn) and [nginx](http://nginx.org/). This configuration should roughly match the production environment.

## Installation

Clone the repo and run the following commands:

```
git submodule update --remote --recursive --init
PORT=80 TITLE=filedrop TURN_SECRET=CHANGE_ME docker-compose up --build --force-recreate
```

Make sure docker and docker-compose are installed and your user is in the docker group. In case another reverse proxy is used make sure to change the default port (from 80) and to add the `X-Forwarded-For` header with client's IP address.

TURN uses TCP port 3478 and UDP ports 49152-65535.

## Environment variables

| Variable      | Default value | Description                                         |
| ------------- | ------------- | --------------------------------------------------- |
| `PORT`        | `80`          | Reverse proxy port.                                 |
| `TITLE`       | `filedrop`    | Application title.                                  |
| `TURN_SECRET` | null          | TURN secret. Should be set to a random, long value. |

## HTTPS setup

### Setup with a reverse proxy before nginx (easy)

1. Configure your reverse proxy to proxy requests to `127.0.0.1:PORT` and then follow your usual instructions for using SSL certificates with said proxy.
2. Rebuild the application.
3. Make sure the TURN server can be connected to from the outside.

#### Nginx configuration example

More details available here: https://www.nginx.com/blog/websocket-nginx/

```nginx
worker_processes auto;

events {
  worker_connections 1024;
}

http {
  upstream filedrop {
    server 127.0.0.1:5000; # 5000 = PORT
  }

  map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
  }

  # ...

  server {
    listen 80;
    # server_name should be configured here.
    # HTTPS should be configured here. (certbot will handle this for you, if you're using Let's Encrypt.)

    # ...

    location / {
      proxy_pass http://filedrop;
      proxy_http_version 1.1;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header Host $host;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection $connection_upgrade;
    }
  }
}
```

### Setup without another reverse proxy (difficult)

There are three things that need to be adjusted for HTTPS support.

1. Certificates must be configured in `conf/nginx/nginx.conf`. There are several guides available for configuration of nginx in docker with Let's Encrypt:
   - https://mindsers.blog/post/https-using-nginx-certbot-docker/
   - https://pentacent.medium.com/nginx-and-lets-encrypt-with-docker-in-less-than-5-minutes-b4b8a60d3a71
2. Port 443 needs to be forwarded for the nginx service.
   1. The following needs to be added in the `services` -> `nginx` -> `ports` dictionary in the `docker-compose.yml` file:
      - `- "${SECURE_PORT}:443"`
   2. Then for further builds, this command must be used instead:
      - `SECURE_PORT=443 PORT=80 TITLE=filedrop TURN_SECRET=CHANGE_ME docker-compose up --build --force-recreate`
3. Configure `REACT_APP_SERVER` in the `services` -> `web` -> `build` -> `args` dictionary in `docker-compose.yml` to use the `wss://` protocol instead of `ws://`, and to use the domain of the certificate you're using.
4. Rebuild the application.
5. Make sure the TURN server can be connected to from the outside.
