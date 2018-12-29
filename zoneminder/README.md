# Zoneminder

[Zoneminder](https://github.com/ZoneMinder/zoneminder) is an application for
managing/using CCTV and security cameras.

This directory contains instructions for setting up ZM on a box within docker.

Data is written to `/data/zoneminder/`.


## Installation

### Install docker

Install Docker CE on the machine (if it not already). Instructions for Ubuntu can
be found [here](https://docs.docker.com/install/linux/docker-ce/ubuntu/) (see
the "Install using the repository" section).

Add your user to the docker group

```
sudo usermod -aG docker $USER
```

Log out, and log back in again for the new group to take effect.

Run `docker run --rm hello-world` to test that docker is working.

### Run the docker containers

```sh
# Create a docker network for ZM to talk to MySQL over
docker network create zm-network

# Run an instance of the MySQL database in a docker container
docker run -d \
  -v /data/zoneminder/mysql:/var/lib/mysql \
  -e TZ=Europe/London \
  -e MYSQL_USER=zmuser \
  -e MYSQL_PASSWORD=zmpass \
  -e MYSQL_DATABASE=zm \
  -e MYSQL_ROOT_PASSWORD=mysqlpsswd \
  -e MYSQL_ROOT_HOST=% \
  --net zm-network \
  --name zm-mysql \
  mysql/mysql-server:5.7

# Now wait for a few seconds for the database to start...

# Run an instance of the ZM application
docker run -d \
  --shm-size=4096m \
  -v /var/empty:/var/empty \
  -v /data/zoneminder/backups:/var/backups \
  -v /data/zoneminder/zoneminder:/var/cache/zoneminder \
  -e TZ=Europe/London \
  -e ZM_DB_HOST=zm-mysql \
  --net zm-network \
  --name zm-app \
  -p 80:9000 \
  quantumobject/docker-zoneminder
```

Now the ZM application is running on [localhost:9000/zm](localhost:9000/zm)

## Usage

The MySQL and ZM containers can be stopped like so:

```sh
docker stop zm-app
docker stop zm-mysql
```

The MySQL and ZM containers can be started like so:

```sh
docker start zm-mysql
# Wait a few seconds for the database to start...
docker start zm-app
```
