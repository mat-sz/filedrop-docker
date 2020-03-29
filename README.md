# filedrop-docker

A docker-compose setup for filedrop. Contains: [filedrop-web](https://github.com/mat-sz/filedrop-web), [filedrop-ws](https://github.com/mat-sz/filedrop-ws), [coturn](https://github.com/coturn/coturn) and [nginx](http://nginx.org/). This configuration should roughly match the production environment.

## Installation

Clone the repo and run the following commands:

```
git submodule update --recursive --init
HOST=localhost PORT=80 TITLE=filedrop TURN_SECRET=CHANGE_ME docker-compose up
```

Make sure docker and docker-compose are installed and your user is in the docker group. In case another reverse proxy is used make sure to change the default port (from 80) and to add the `X-Forwarded-For` header with client's IP address.

## Environment variables

| Variable      | Default value | Description                                         |
| ------------- | ------------- | --------------------------------------------------- |
| `HOST`        | `localhost`   | Domain/external IP for the application.             |
| `PORT`        | `80`          | Reverse proxy port.                                 |
| `TITLE`       | `filedrop`    | Application title.                                  |
| `TURN_SECRET` | null          | TURN secret. Should be set to a random, long value. |
