# Docker/Compose Debian Setup

[Docker](https://docs.docker.com/)
is a lightweight virtualization environment that is gaining fast 
acceptance in the development and deployment communities. With 
Docker -- we can document the construct of a specific image
so that it can be reliably duplicated in other environments and
layer them so that that images can be layered on top one 
another (e.g., Debian Image `<-` Ruby Image)

[Docker Compose](https://docs.docker.com/compose/overview/) is a
convenient way to define a collection of individual Docker instances
so that we can maintain a network of database, web server, and other
potential resources all running in their own containers in our local
development environment.

These instructions and this course will not teach you Docker or
Docker Compose -- just as much as the OS-specific instructions were
not provided to teach you Windows, MacOS, etc. The purpose of
providing these instructions are for students that already have 
some familiarity with Docker and are looking for an example of 
how to use it in class with Rails, MongoDB, PostgreSQL, etc.

The OS platforms used to deploy the software within Docker is Debian-based
(`Linux ce9c4f126ef9 4.4.27-moby #1 SMP Wed Oct 26 14:21:29 UTC 2016 x86_64 GNU/Linux`)
-- so straight Ubuntu users can also get a sense of the setup
required as well by looking at the 
[Ruby](https://github.com/docker-library/ruby/blob/1913ca0a1268c9a3d18ca5771a85ec3428c9cb3d/2.2/Dockerfile)
root Dockerfile(s) and the contents of the Docker file built here.

When we install Docker, we intend to setup the following virtual machines
* postgres
* mongo 
* railsvm
* rake
* test
* selenium

`postgres`, `mongo`, and `railsvm` are what they sound like -- they
are stand-alone instances of the PostgreSQL and MongoDB databases
and Rails server. `rake` is a variant of `railsvm` that is designed
to run the entire test suite without any interaction. `test` is 
a variant of `rake` that is designed for debugging tests by providing
a VNC server. The `selenium` service runs a standalone chrome or 
firefox browser exposed through a VNC server.

**Note**: `railsvm`, `rake`, and `test` expose a few of the same 
TCP ports and are not meant to be run concurrently at this time.
`railsvm` is what we use when we want to interact with the application
as an application. `rake` is what we use on the build server (e.g., 
Jenkins), and `test` is what we use to locally debug tests that are
failing on the build server. 

**Note**: Although we could use this environment as our primary
development environment -- it would not be as performant as a native
environment run locally.

## Install Docker

As stated, these instructions and this course are not meant to teach
Docker from scratch. The installation of Docker is a pre-requisite
to continuing with these Docker-based instructions and will not be
covered here. It is assumed that if we are going that path that we
already have Docker installed or will take care of that on our own.

* [Download and Install Docker](https://docs.docker.com/)

**Note**: The only thing I will add beyond the Docker
instructions/requirements is that you have at least 10-20GB of free
disk space to hold images (more would be preferable). I have gotten
by with much less in the past, but had to constantly perform garbage
collection on outdated image files.

## Setup Images

Much of what is listed below was gisted from the 
[Docker Compose Rails Quickstart](https://docs.docker.com/compose/rails/)
and augmented for use in the course.  As will most Docker installations,
we can boostrap our local environment pretty easily by extending
an existing repository image. We will show this way first and then
show what we extended.

1. Add a Dockerfile for the local application. This parent Dockerfile
requires the application have a Gemfile and Gemfile.lock file in the 
root directory.

    ```dockerfile
    # Dockerfile

    FROM ejavaguy/capstone_rails-onbuild:latest
    ```

2. Add a boostrap Gemfile and Gemfile.lock to the application

    * Gemfile

        ```ruby
        # Gemfile

        source 'https://rubygems.org'
        gem 'rails', '4.2.6'
        gem 'rails-api'
        ```

    * Gemfile.lock

        ```shell
        $ touch Gemfile.lock
        ```

3. Add a `docker-compose.yml` for the local installation.
It might be easier to grab a copy from a complete application 
[example in the git repository](://github.com/jhu-ep-coursera/capstone_docker/blob/master/sample_app/docker-compose.yml).

    * Initially populate the `docker-compose.yml` file with the 
    core operational services: `postgres`, `mongo`, and `railsvm`.

        ```yaml
        # docker-compose.yml

        version: '2'
        services:
          postgres:
            image: postgres
            environment:
              - POSTGRES_PASSWORD
          mongo:
            image: mongo
          railsvm:
            build: .
            command: bundle exec rails s -p 3000 -b '0.0.0.0'
            volumes:
              - .:/myapp
            ports:
              - "3000:3000"
            depends_on:
              - postgres
              - mongo
            environment:
              - POSTGRES_USER=postgres
              - POSTGRES_HOST=postgres
              - POSTGRES_PASSWORD
              - MONGO_HOST=mongo:27017
        ```
        * Environment variables define volatile configurations
        and sensitive credentials that we should not check into 
        Git with our source code. These are many of the same 
        environment variables we will define within our Heroku
        shell.

    * Build the default instance with the boostrap Gemfile using 
    `docker-compose build`. This will download the base Docker 
    image(s) and then run `bundle` to install necessary gems in 
    the local instance. We will execute the `build` command each
    time we change the Gemfile.

        ```ruby
        $ docker-compose build
        mongo uses an image, skipping
        postgres uses an image, skipping
        Building railsvm
        Step 1 : FROM ejavaguy/capstone_rails-onbuild:latest
        # Executing 3 build triggers...
        Step 1 : COPY Gemfile $APP_HOME
        Step 1 : COPY Gemfile.lock $APP_HOME
        Step 1 : RUN bundle install
         ---> Running in 67f0f3e3d864
        Fetching gem metadata from https://rubygems.org/..........
           ...
        Bundle complete! 2 Gemfile dependencies, 36 gems now installed.
        Bundled gems are installed into /usr/local/bundle.
         ---> 7d2896187733
        Removing intermediate container a5def8667391
        Removing intermediate container bc7d18df658c
        Removing intermediate container 67f0f3e3d864
        Successfully built 7d2896187733
        ```

        **Note**: One reason the base image(s) are so large is 
        that they come with all software and gems pre-loaded
        for use with the course. The bundle command should ideally
        run quickly because most gems can be located within the 
        built image.


## Base Image: `capstone_rails`

1. The image is based of the baseline `ruby:2.2` version from the 
Docker repository. Version `ruby:2.3` is also available but to 
earlier version was used for some consistency with the Windows 
platform decisions.

    ```dockerfile
    # Dockerfile

    FROM ruby:2.2
    ```

2. Firefox (iceweasel) is installed for use as an image viewer
for Capybara screenshots to debug failing tests

    ```dockerfile
    # Dockerfile

    # Install firefox (aka iceweasel)
    RUN apt-get update -qq && \
      DEBIAN_FRONTEND=noninteractive apt-get install -qq -y \
        iceweasel
    ```

3. VNC server is installed in order to be able to see the 
browser

    ```dockerfile
    # Dockerfile

    # Install LXDE and VNC server to be able to see firefox
    RUN apt-get update && \
      DEBIAN_FRONTEND=noninteractive apt-get install -qq -y \
        lxde-core \
        lxterminal \
        tightvncserver
    ENV USER root
    ```

4. Phantomjs is downloaded and installed to provide support
for headless poltergeist testing

    ```dockerfile
    # Dockerfile

    # install phantomjs - download from https://bitbucket.org/ariya/phantomjs/downloads
    ADD https://bitbucket.org/ariya/phantomjs/downloads/phantomjs-2.1.1-linux-x86_64.tar.bz2 .
    RUN tar -xjf phantomjs-2.1.1-linux-x86_64.tar.bz2 
    RUN mv ./phantomjs-2.1.1-linux-x86_64/bin/phantomjs /usr/local/bin
    RUN rm -rf ./phantomjs-2.1.1-linux-x86_64
    RUN phantomjs --version
    ```

5. Core Rails support libraries are installed to work with postgres
and provide a javascript runtime

    ```dockerfile
    # Dockerfile

    # install postgres and nodejs Rails support packages
    RUN apt-get update -qq && \
      DEBIAN_FRONTEND=noninteractive apt-get install -qq -y \
        build-essential \
        libpq-dev \
        postgresql-client \
        nodejs
    ```

6. ImageMagick was installed in the root image, but fails unless
we add ghostscript fonts

    ```dockerfile
    # Dockerfile

    # install ghostscript fonts for already installed imagemagick
    RUN apt-get update -qq && \
      DEBIAN_FRONTEND=noninteractive apt-get install -qq -y \
        ghostscript
    ```

6. Some repository cleanup is performed after the package installs

    ```dockerfile
    # Dockerfile

    # cleanup repositories
    RUN rm -rf /var/lib/apt/lists/*
    ```

7. A directory is created within the VM to host the application
files. **Note**: The same directory is replaced using a 
shared filesystem mount at runtime.

    ```dockerfile
    # Dockerfile

    # define a directory within the VM for the application
    ENV APP_HOME /myapp
    RUN mkdir $APP_HOME
    WORKDIR $APP_HOME
    ```

8. The Gemfile and Gemfile.lock is copied into the image and the
bundler is run against that definition. The source directory for
the root image happen to use a fully populated Gemfile that is
consistent with what will should end up with at the end of the
course. That means that many of the gems will be later resolved
locally from what we have installed here in this base image.

    ```dockerfile
    # Dockerfile

    # VM will be rebuilt from this point when these files change
    ADD Gemfile $APP_HOME
    ADD Gemfile.lock $APP_HOME
    RUN bundle config --global --jobs 4
    RUN bundle install
    ```

9. Set the display for the first (:0) display and copy in a 
VNC startup script.

    ```dockerfile
    # Dockerfile


    # copy this last since has the most likely changes
    ENV DISPLAY :0
    COPY vnc.sh /usr/local/bin
    ```

    The contents of `vnc.sh` is as follows. The password for the 
    VNC server is set using the `VNC_PASSWORD` environment 
    variable passed in at runtime.

    ```shell
    #!/bin/bash

    set -x

    # Remove VNC lock (if process already killed)
    rm -f /tmp/.X0-lock /tmp/.X11-unix/X0

    # set the VNC password using a value from environment
    #echo VNC_PASSWORD=${VNC_PASSWORD}
    printf "${VNC_PASSWORD}\n${VNC_PASSWORD}\n" | vncpasswd

    # Run VNC server
    vncserver :0 -geometry 1280x800 -depth 24

    #uncomment to debug or to keep VM running
    #tail -F /root/.vnc/*.log
    ```

10. Finally -- the ports used by the VM (Rails and VNC) are being
advertized.

    ```dockerfile
    # Dockerfile

    EXPOSE 3000 5900
    ```

## Start/Stop Servers

* To start our database servers, we can execute the `up` command issuing
the `-d` option for detached

    ```shell
    $ docker-compose up -d postgres mongo
    Starting capstonedemoapp_mongo_1
    Starting capstonedemoapp_postgres_1
    ```

* To execute a specific command against a service, we can use the `run`
command and name the server to execute against

    ```shell
    $ docker-compose run railsvm ruby -e "puts ENV['MONGO_HOST']"
    mongo:27017
    ```

* To view the logs of a running instance, we can 

    ```shell
    $ docker-compose logs postgres
    ```

* To stop an instance, we can use the `stop` command

    ```shell
    $ docker-compose stop postgres
    Stopping capstonedemoapp_postgres_1 ... done
    ```

* To bring everything down gracefully, we can use the `down` command

    ```shell
    $ docker-compose down
    Stopping capstonedemoapp_mongo_1 ... done
    Removing capstonedemoapp_mongo_1 ... done
    Removing capstonedemoapp_postgres_1 ... done
    Removing network capstonedemoapp_default
    ```

## Manage Images

For those not as familiar with Docker and the images created, some
familiarity with the following commands should come in handly

* listing active container instances

    ```shell
    $ docker ps
    ```

* listing active and in-active container instances

    ```shell
    $ docker ps -a
    ```

* stopping active container instance

    ```shell
    $ docker stop [image name|container id]
    ```

* removing inactive container instances

    ```shell
    $ docker rm [image name|container id]
    ```

* listing images

    ```shell
    $ docker images
    ```

* removing an image (with no active instances)

    ```shell
    $ docker rmi [repository name|image id]
    ```

## Summary

We now have our database and Rails server VM instances in place
and are now at a point where we are ready to start creating an
application to deploy in the VM.

It is assumed that the information contained within this 
section is informational and that we will not be using 
Docker for our primary development environment. However,
if we do choose to use it for one reason or another (e.g., 
cannot install ImageMagick locally), then please remember
two things:

* The code will execute within the VM and must have access
to resources (e.g., gems, files, etc) that have either 
been persisted inside the virtual machine during the `build`
or is accessible through the shared file system mount.

* Any command we execute to manipulate the local file 
system through the shared file system mount, must be 
prefixed with 

    ```shell
    $ docker-compose run railsvm
    ```

    Example:

    ```shell
    $ docker-compose run railsvm rails-api new ...
    ```
