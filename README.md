# MISP Docker

[![Build status](https://travis-ci.org/seanthegeek/misp-docker.svg?branch=master)](https://travis-ci.org/seanthegeek/misp-docker)

A docker Compose project for [MISP](http://www.misp-project.org) ("Malware Information Sharing Platform").

This is a work in progress rewrite of the [MISP Docker (XME edition)](https://github.com/MISP/misp-docker) to resolve longstanding bugs and add new features.

*Warning*: Please see the [Issues](https://github.com/seanthegeek/misp-docker/issues) tab for known issues.

## Features

* Creates separate Docker containers for the database, web server, and reverse proxy
* Reconfiguration of the base URL in `config.php`
* Generation of a new salt in `config.php`
* Generation of a self-signed certificate
* Optimization of `php.ini` to match the MISP recommended values
* Creation of the MySQL database
* Generation of the admin PGP key

## Setup

1. Add your `misp.crt` and `misp.key` files to the `./proxy/ssl` folder
2. Update `server_name` in `default.conf` file (will implement ENVIRONMENT VARIABLE in the future)

### Building your image

#### Fetch files

```bash
git clone https://github.com/seanthegeek/misp-docker
cd misp-docker
```

## Edit the .env file

```bash
cp template.env .env
nano .env
```

## Build the containers

```bash
docker-compose build
```

## Run the containers

``` bash
docker-compose up -d
```
