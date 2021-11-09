# Docker web Toolkit

This toolkit is set of tools aiming to simplify the execution of local containerized applications.

It contains the 4 following pieces of software:

- [Traefik](https://doc.traefik.io/traefik/): a reverse proxy listening on port 80 and 443 for routing requests to specific containers.
- [Ghosts](https://github.com/lobre/ghosts): tool to automatically register local domain names into the native OS `hosts` file.
- [Adminer](https://www.adminer.org/): tool to manage all databases connected in a same docker network.
- [Mailhog](https://github.com/mailhog/MailHog): SMTP server to catch and browse all mails sent from the local application (development environment).

## Set up

This project has to be started only once as it will automatically restart when the machine reboots.

First, you have to create a `.env` file at the root of the project, and add the different variables (example):

    $ cat .env
    traefik_url=traefik.local
    ghosts_url=ghosts.local
    adminer_url=adminer.local
    mailhog_url=mailhog.local

You can find an example in `.env.example` file.

### Development environment on Linux

You can start the docker stack with:

    $ docker-compose up -d

This command use the `docker-compose.yml` and the `docker-compose.override.yml` files.

Once it has started you can browse [ghosts.local](https://ghosts.local) to find links to exposed containers throught the proxy.

### Development environment on Windows

First, you have to install [Docker for Windows](https://docs.docker.com/docker-for-windows/install/).

**Warning**: Install Docker with Linux containers and not Windows containers.

To let the ghosts container edit the `C:\Windows\System32\drivers\etc\hosts`, you need to update the permissions of the file to let the current user edit without Administrator rights (see the [ghosts documentation](https://github.com/lobre/ghosts/blob/master/README.md)).

Then you can launch:

    $ docker-compose -f docker-compose.yml -f docker-compose.override.yml -f docker-compose.override-win.yml up -d

Once it has started, you can browse [ghosts.local](https://ghosts.local) to find links to exposed containers through the proxy.

### Production environment on Linux

You have to add prod variables into the `.env` file:

    $ cat .env
    traefik_email=traefik@example.com
    traefik_auth=user:xxxx
    ghosts_auth=user:xxxx
    adminer_auth=user:xxxx

You can find an example in `.env.example` file.

Then, you can launch:

    $ docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d

Once it has started, you can browse [your_ghosts_url](https://your_ghosts_url.com) to find links to exposed containers through the proxy.

## Add connected services

To add services connected with the toolkit, you have to add labels into the service declaration:

    ...

    services:
        your_app:
            ...
            networks:
                ...
                - server_proxy
            labels:
                ...
                - traefik.enable=true
                - traefik.http.routers.<your_app_name>_secure.entrypoints=websecure
                - traefik.http.routers.<your_app_name>_secure.rule=Host(`${your_app_url}`) # to link url to this service
                - traefik.http.routers.<your_app_name>_secure.tls=true

                # just in prod env
                - traefik.http.routers.<your_app_name>_secure.tls.certresolver=letsencrypt

                - ghosts.host=${your_app_url}
                - ghosts.category=~Applications # category on ghosts page
                - ghosts.logo=your_app_logo.png # logo on ghosts page
                - ghosts.description=Your app description. # description on ghosts page
                - ghosts.proto=https

You can define the port exposed by your service with this label:

    - traefik.http.services.<your_app_name>.loadbalancer.server.port=<your_port>

If your services expose just one port, you can omit this label.

## How to configure MailHog

The application container have to be link with the `server_mail` network.

Then, the mailhog address will be `mailhog` and the port is `1025`.

## How to use Adminer

The database container have to be link with the `server_db` network.

You can access to your database on the Adminer page with this credentials:

    System: your database system.
    Server: the name of the container of the database.
    Username: the username of the database.
    Password: the password of the database.
    Database: the name of the database.