# docker-otrs
Unofficial [OTRS 4 Free](http://www.otrs.com/software/) docker image. This repository contains the 
Dockerfiles and all other files needed to build and run the container. 

We also include a MariaDB Dockerfile for a pre-configured image with the database [required settings](http://otrs.github.io/doc/manual/admin/stable/en/html/installation.html) 
needed to be run with OTRS. If you link to this image, use the link alias name (ej: mariadb) as the database name and the when running the installer.

The OTRS image doesn't include a SMTP service, decoupling applications into multiple containers makes it 
much easier to scale horizontally and reuse containers. If you don't have access to a SMTP server, you 
can link against this [SMTP relay](https://github.com/juanluisbaptiste/docker-postfix) container instead.

These images are based on the [official CentOS 7.0 images](https://registry.hub.docker.com/_/centos/) and 
include the latest OTRS version. Older images are tagged with the OTRS version they run. 

Mod_perl is enabled by default.

### Build instructions

We use `docker-compose` to build the images. Clone this repo and then:

    cd docker-otrs
    sudo docker-compose build

This command will build all the images and pull the missing ones like the [SMTP relay](https://github.com/juanluisbaptiste/docker-postfix).

You can also find a prebuilt image from [Docker Hub](https://registry.hub.docker.com/u/juanluisbaptiste/otrs/), 
which can be pulled with this command:

    sudo docker pull juanluisbaptiste/otrs:latest

### How to run it

By default, when the container is run it will load a default vanilla OTRS 
install that is ready to be configured as you need. However, you can load 
a backup or run the installer by defining one of these env variables:

* `LOAD_BACKUP=yes` Will look for otrs-latest.sql and Config.pm.latest at 
/opt/otrs/docker/backup and load them.
* `DEFAULT_INSTALL=no` Will run the installer.

There are also some other environment variables that can be set to customize
the default install:

* `OTRS_HOSTNAME` Sets the container's hostname.
* `OTRS_ADMIN_EMAIL` sets the admin user email.
* `OTRS_ORGANIZATION` Sets the organization name (ex: MyCompany Ltd.)
* `OTRS_DB_PASSWORD` otrs user database password. Default password is `changeme`.
* `OTRS_ROOT_PASSWORD` root@localhost user password. Default password is `changeme`.

For testing the containers you can bring them up with `docker-compose`:

    sudo docker-compose build
    sudo docker-compose up

This will bring up all needed containers, link them and mount volumes according 
to the `docker-compose.yml` configuration file. The default database password is 
`changeme`, to change it edit the `docker-compose.yml` file and change the 
`MYSQL_ROOT_PASSWORD` environment variable on the `mariadb` image definition before running `docker-compose`.

To start them individually (not recommended, better use `docker-compose`):

    docker run -d -P --name mariadb -v /var/lib/mysql \ 
        -e MYSQL_ROOT_PASSWORD=xxxxx \ 
        juanluisbaptiste/otrs-mariadb
    docker run -d -P --name postfix \ 
        -e SMTP_SERVER=xxxxxx \
        -e SMTP_USERNAME=xxxx \ 
        -e SMTP_PASSWORD=xxxx \ 
        juanluisbaptiste/postfix 
    docker run -d -p "80:80" --name otrs \
        --link="mariadb:mariadb" \
        --link="postfix:postfix" \
        juanluisbaptiste/otrs
