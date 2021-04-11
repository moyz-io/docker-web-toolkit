# Docker Web Toolkit

This toolkit is set of tools aiming to simplify the execution of local containerized applications.

It contains the 4 following pieces of software.

- [Traefik](https://doc.traefik.io/): a reverse proxy listening on port 80 and 443 for routing requests to specific containers.
- [Ghosts](https://github.com/lobre/ghosts): tool to automatically register local domain names into the native OS `hosts` file.
- [Adminer](https://www.adminer.org/): tool to manage all databases connected in a same docker network.
- [MailHog](https://github.com/mailhog/MailHog): SMTP server to catch and browse all mails sent from the local application.

## Set up

This project has to be started only once as it will automatically restart when the machine reboots.

First, you have to create a `.env` file at the root of the project, and add the different variables (example):

    $ cat .env
    traefik_url=traefik.local
    ghosts_url=ghosts.local
    adminer_url=adminer.local
    mailhog_url=mailhog.local

### Development environment on Linux

You can start the docker stack with:

    docker-compose up -d

This command use the `docker-compose.yml` and the `docker-compose.override.yml` files.

Once it has started, you can browse [ghosts.local](https://ghosts.local) to find links to exposed containers through the proxy.

### Development environment on Windows

First, you have to install [Docker for Windows](https://docs.docker.com/docker-for-windows/install/).

**Warning:** Install Docker with Linux containers and not Windows containers.

Then, you can launch:

    docker-compose -f docker-compose.yml -f docker-compose.override.yml -f docker-compose.override-win.yml up -d

Once it has started, you can browse [ghosts.local](https://ghosts.local) to find links to exposed containers through the proxy.

### Production environment on Linux

You have to add others variables into the `.env` file:

    $ cat .env
    traefik_email=traefik@example.com
    traefik_auth=user:xxxx
    ghosts_auth=user:xxxx
    adminer_auth=user:xxxx
    mailhog_auth=user:xxxx

Then, you can launch:

    docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d

Once it has started, you can browse [your_ghosts_url](https://your_ghosts_url.com) to find links to exposed containers through the proxy.

## Add connected services

To add services connected with the toolkit, you have to add labels into the service declaration:

    ...

    services:
        your_app:
            ...
            labels:
                ...
                - traefik.enable=true # to active traefik with this app
                - traefik.http.routers.your_app_http.rule=Host(`${your_app_url}`) # to link url to this service
                - traefik.http.routers.your_app_http.middlewares=redirect-to-https@file # redirection from web to websecure
                - traefik.http.routers.your_app_http.entrypoints=web

                - traefik.http.routers.your_app_https.rule=Host(`${your_app_url}`) # to link url to this service (same url than web)
                - traefik.http.routers.your_app_https.entrypoints=websecure
                - traefik.http.routers.your_app_https.tls=true # active websecure

                - ghosts.host=${your_app_url} # your app url
                - ghosts.category=~Applications # category on ghosts page
                - ghosts.logo=your_app_logo.png # logo on ghosts page
                - ghosts.description=Your app description # description on ghosts page
                - ghosts.proto=https

## How to configure Mailhog

## How to use Adminer

The database container have to have the `server_db` network.

    System: your database system.
    Server: the name of the container of the database.
    Username: the username of the database.
    Password: the password of the database.
    Database: the name of the database.