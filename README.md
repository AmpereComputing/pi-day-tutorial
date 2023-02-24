# Deploying a LAMP stack using docker-compose

This project will enable you to get started with a starter multi-container LAMP stack using docker
containers. We will start 3 containers, each with a specific role:

1.  `nginx`: We map port 80 of `localhost` to port 80 of the `nginx` container. This container will
    proxy for the entire application. This will enable us to modify the architecture of our application
    without changing how our users interact with it. For example, behind a proxy, we might load balance
    across multiple application servers, or add a reverse proxy cache to avoid hitting the application
    for static resources.
2.  `wordpress`: This is a basic Apache2/PHP container with the latest version of Wordpress pre-installed.
    We add configuration options for the Wordpress container for the database connection to a separate
    database node. For test purposes, we map port 80 of the Wordpress container to port 8080 of the host.
3.  `mysql-server`: This container will serve as our database. The database will be initialized with an
    empty database and a pre-configured user through environment variables.

Each of these containers is added to a bridge network for the application. These containers can reach
each other by name (`db`, `wordpress`, and `proxy`), but from the host, we cannot connect to them,
except via the `docker` command. For example, if I try to point my web browser at the `wordpress`
container, I cannot connect to it. 

## Prerequisites

Throughout, we assume that you are running on a modern Linux distribution. The author uses the RHEL 
family of operating systems - CentOS Streams, AlmaLinux, Rocky Linux, Oracle Linux, but any
platform-specific commands will be flagged. If you are using a Debian-family distribution, you will
need to use an equivalent on those platforms.

We also assume that you have installed Docker (docker.io) and
[docker-compose](https://github.com/docker/compose/releases). The instructions should work with
podman and podman-compose, but we have not tested them.

We will also use `git` to check out the code from github.

We will want to run docker-compose as an unprivileged user, so we have to add our user to
the `docker` group.

    usermod -aG docker $USER

After running this, you need to ensure that you relaunch a login shell for the change to take effect.
You can do this explicitly with:

    sudo -s -u $USER

but for the change to take effect permanently in your instance, you will need to restart the `sshd`
service. On the RHEL family, this means:

    sudo systemctl restart sshd

The next time you connect to your instance with ssh, you should see `docker` group in the output of
`groups` for your user.

To test that you now have Docker correctly installed and configured, run:

    docker run hello-world

You should see a Docker container being downloaded from dockerhub, and a message confirming that your
installation appears to be running correctly.

## Getting started

Now that we have things set up, check out this project:

    git clone https://github.com/dneary/docker-lamp.git
    cd docker-lamp

We need a few directories to be created that do not yet exist:

    mkdir -p wordpress mysql/wordpress mysql/logs mysql/initdb

We can now copy a database back-up from another Wordpress site into `mysql/initdb` to initialize the
database when we start our stack. If you don't already have a back-up, don't worry about it, we will
go through the Wordpress installation process.

We should now be ready to run:

    docker-compose up -d

This will process the docker-compose.yml file in this directory, and download three containers: `nginx`, 
`wordpress`, and `mysql-server`. We start these containers as the services `proxy`, `wordpress`, and `db`,
and they are all added to a bridge network created for the project. Since we do not (yet) have a domain name,
we do not enable and configure `https` using SSL certificates. If all goes well, you should now be able to
go to `http://localhost:80` and start the Wordpress installation process.
