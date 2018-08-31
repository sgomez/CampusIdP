# Dockerized Campus IdP platform

This document briefly describes dockerized _Campus IdP_ platform. It has been developed and tested using `Docker version 18.03.1-ce` and `docker-compose version 1.21.2` so make sure you are using at least the versions mentioned.

The primary way to run this is to use `docker-compose` command, but prior to this one has to prepare a few configuration files and load a few variables into the terminal.

## debian:stretch-slim

Using `debian:stretch` official Debian Docker image produces an image 339MB in size. There is also `debian:stretch-slim`, which is 45 MB smaller compared to `debian:stretch`. In case such saving is reasonable for you, you can switch to _slim_ image. In that case you need to change first line of `Dockerfile` and also put `mkdir -p /usr/share/man/man1` right before `apt-get install` command due to a [reported bug][].

| Docker image          | Size  |
| --------------------- | -----:|
| `debian:stretch`      | 339MB |
| `debian:stretch-slim` | 286MB |
| `mysql:5`             | 372MB |
| `mariadb:10.1`        | 407MB |

Althoug, MySQL container for storing `persistent-id` (aka `eduPersonTargetedID` attribute) consumes _372MB_, saving 53MB could make a difference for someone.

It could be saved 48MB by deleting `/opt/shibboleth-identity-provider-$SHIBBOLETH_VERSION/` directory which is not neccessary after it has been installed using `install.sh` script. However, that assumes the installation is run from the `Dockerfile` not `docker-entrypoint.sh`. And I am sure it is not worth it.

## MySQL or MariaDB

Although the current `docker-compose.yml` deploys MySQL (`mysql:5`), it has been tested with MariaDB (`mariadb:10.1`) as well. Just tweak `docker-compose.yml` as follows:

```
  db:
    image: mariadb:10.1
    hostname: mariadb
```

## Run an IdP

As the first step, you have to prepare `env.conf` file. You should not touch `JETTY_VERSION` and `SHIBBOLETH_VERSION` variables unless you know what you are doing.

### env.conf

This file holds all the information required by an IdP. There are passwords mostly, however, you can define _scope_ and _entityID_ for the IdP etc.

```
JETTY_VERSION=9.3.24.v20180605
JETTY_CERT_KEY=iemooP4mu3neuPhiequi

SHIBBOLETH_VERSION=3.3.3
SHIBBOLETH_SCOPE=idp.example.org
SHIBBOLETH_ENTITYID=https://idp.example.org/idp/shibboleth
SHIBBOLETH_HOSTNAME=idp.example.org
SHIBBOLETH_PID_SALT=ceek9xa0Ahmoh0uyiwah

MYSQL_ROOT_PASSWORD=shie9aez5Ahzakah9aen
MYSQL_DATABASE=shibboleth
MYSQL_USER=shibboleth
MYSQL_PASSWORD=miehaiph3chohghoaXah
```

### idp.conf

In this file, which is loaded automatically by `docker-compose.yml`, all "internal" configuration for IdP is defined. These variables are not required for building the image, these are required only when `docker-entrypoint.sh` is run to configure installed IdP.

```
LDAP_AUTHENTICATOR=bindSearchAuthenticator
LDAP_LDAPURL=ldaps://ldap.example.org:636
LDAP_USESTARTTLS=false
LDAP_USESSL=true
LDAP_SSLCONFIG=certificateTrust
LDAP_BASEDN=ou=people,dc=example,dc=org
LDAP_SUBTREESEARCH=false
LDAP_BINDDN=uid=shibboleth,ou=special users,dc=example,dc=org
LDAP_BINDDNCREDENTIAL=taiSh9aishaimoo7tiey

PERSISTENTID_SOURCEATTRIBUTE=uid
PERSISTENTID_SALT=SWmGA5tfBJMI+PZHyKbp9D/CA9rw+omRcFcNw4XftbGDVduF
```

### Loading variables

Now you must load all the variables in the current shell.

```bash
export $(cat conf/idp.example.org/env.conf | xargs)
```

### Start your IdP

And now it is possible to run `docker-compose` command. It is a very good idea to specifi "a project name" using `-p` parameter. I would use domain name or server host name as a value.

```bash
docker-compose -p example up -d --build
```

## Stop an IdP

To shut down the IdP including database, run:

```bash
docker-compose -p example down
```

### Stop and deleting everything

To stop and delete running container, delete all built images and data volumes, run the following command:

```bash
docker-compose -p example down --rmi all -v --remove-orphans
```

You can override all variables by loading a different file. You can also simply unset variables or even simpler close the current shell.

```bash
unset JETTY_VERSION
unset JETTY_CERT_KEY

unset SHIBBOLETH_VERSION
unset SHIBBOLETH_SCOPE
unset SHIBBOLETH_ENTITYID
unset SHIBBOLETH_HOSTNAME
unset SHIBBOLETH_PID_SALT

unset MYSQL_ROOT_PASSWORD
unset MYSQL_DATABASE
unset MYSQL_USER
unset MYSQL_PASSWORD
```

[reported bug]: https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=863199
