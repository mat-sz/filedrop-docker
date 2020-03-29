# filedrop-docker

A docker-compose setup for filedrop. Contains: [filedrop-web](https://github.com/mat-sz/filedrop-web) and [filedrop-ws](https://github.com/mat-sz/filedrop-ws). This configuration should roughly match the production environment.

## Installation

Clone the repo and run the following commands:

```
git submodule update --recursive --init
docker-compose up
```

Make sure docker and docker-compose are installed and your user is in the docker group.

## Using custom ports

Please copy the attached `docker-compose.override.example.yml` file to `docker-compose.override.yml` and modify the values as needed.
