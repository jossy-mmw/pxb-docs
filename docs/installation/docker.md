# Running Percona XtraBackup in a Docker container

!!! note

    The following instructions runs Percona XtraBackup 2.4 in a Docker container. The instructions to run Percona XtraBackup 8.0 by the same method are available in the [Percona XtraBackup 8.0 installation documentation](https://docs.percona.com/percona-xtrabackup/8.0/installation/docker.html).

Docker allows you to run applications in a lightweight unit called a container.

You can run *Percona XtraBackup* in a Docker container without installing the product. All required libraries are available in
the container. Being a lightweight execution environment, Docker containers enable creating
configurations where each program runs in a separate container. You may run
*Percona Server for MySQL* in one container and *Percona XtraBackup* in another. Docker images offer a range of options.

Create a Docker container based on a Docker image. Docker images for Percona XtraBackup
are hosted publicly on Docker Hub at `percona/percona-xtrabackup`.

```shell
$ sudo docker create ... percona/percona-xtrabackup --name xtrabackup ...
```

### Scope of this section

This section demonstrates how to backup data
on a Percona Server for MySQL running in another Docker container.

## Installing Docker

Your operating system may already provide a package for **docker**. However,
the versions of Docker provided by your operating system are likely to be
outdated.

Use the installation instructions for your operating system available from the
Docker site to set up the latest version of **docker**.

!!! note

    Docker Documentation:
      * [How to use Docker](https://docs.docker.com/)
      * [Installing](https://docs.docker.com/get-docker/)
      * [Getting started](https://docs.docker.com/get-started/)


## Connecting to a Percona Server for MySQL container

Percona XtraBackup works in combination with a database server. When
running a Docker container for Percona XtraBackup, you can make
backups for a database server either installed on the host machine or running
in a separate Docker container.

To set up a database server on a host machine or in Docker
container, follow the documentation of the supported product that you
intend to use with Percona XtraBackup.

!!! admonition "See also"

    Percona Server for MySQL Documentation:
      * [Installing on a host machine](https://www.percona.com/doc/percona-server/5.7/installation.html)
      * [Running in a Docker container](https://www.percona.com/doc/percona-server/5.7/installation/docker.html)

```shell
$ sudo docker run -d --name percona-server-mysql-5.7 \
-e MYSQL_ROOT_PASSWORD=root percona/percona-server:5.7
```

As soon as Percona Server for MySQL runs, add some data to it. Now, you are
ready to make backups with Percona XtraBackup.

## Creating a Docker container from Percona XtraBackup image

You can create a Docker container based on Percona XtraBackup image with
either **docker create** or **docker run** command. **docker create**
creates a Docker container and makes it available for starting later.

Docker downloads the Percona XtraBackup image from the Docker Hub. If it
is not the first time you use the selected image, Docker uses the image available locally.

```shell
$ sudo docker create --name percona-xtrabackup-2.4 --volumes-from percona-server-mysql-5.7 \
percona/percona-xtrabackup:2.4  \
xtrabackup --backup --datadir=/var/lib/mysql/ --target-dir=/backup \
--user=root --password=mysql
```

With `--name` you give a meaningful name to your new Docker container so
that you could easily locate it among your other containers.

The `--volumes-from` referring to *percona-server-mysql* indicates that you
indend to use the same data as the *percona-server-mysql* container.

Run the container with exactly the same parameters that were used when the container was created:

```shell
$ sudo docker start -ai percona-xtrabackup-2.4
```

This command starts the *percona-xtrabackup* container, attaches to its
input/output streams, and opens an interactive shell.

The **docker run** is a shortcut command that creates a Docker container and then immediately runs it.

```shell
$ sudo docker run --name percona-xtrabackup-2.4 --volumes-from percona-server-mysql-5.7 \
percona/percona-xtrabackup:2.4
xtrabackup --backup --data-dir=/var/lib/mysql --target-dir=/backup --user=root --password=mysql
```

!!! admonition "See also"

    More in Docker documentation
      * [Docker volumes as persistent data storage for containers](https://docs.docker.com/storage/volumes/)
      * [More information about containers](https://docs.docker.com/config/containers/start-containers-automatically/)
