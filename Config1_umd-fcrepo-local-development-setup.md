# umd-fcrepo Local Development Setup

## Introduction

This page provides step-by-step instructions for setting up the
"umd-fcrepo" Docker stack for local development.

## Useful Resources

* [https://github.com/umd-lib/umd-fcrepo](https://github.com/umd-lib/umd-fcrepo)
* [https://github.com/umd-lib/umd-fcrepo-docker](https://github.com/umd-lib/umd-fcrepo-docker)
* [F4: Development Environment](https://confluence.umd.edu/display/LIB/F4%3A+Development+Environment)

## Local Workstation Setup

**Note:** This step only needs to be done once on the host machine.

Edit the "/etc/hosts" file:

```bash
> sudo vi /etc/hosts
```

and add an "fcrepo-local" alias to the "127.0.0.1" entry:

```text
127.0.0.1       localhost fcrepo-local
```

## Quick Start

In the following steps, the existing "latest" version of the Docker images from
the Nexus will be used. See the
["Starting from Scratch"](#starting-from-scratch) section below if you
need to use locally-built or otherwise customized Docker images.

1) Clone the "umd-fcrepo" repository and its submodules:

```bash
git clone --recurse-submodules git@github.com:umd-lib/umd-fcrepo.git
```

1) Switch into the "umd-fcrepo/umd-fcrepo-docker" directory:

```bash
cd umd-fcrepo/umd-fcrepo-docker
```

1) Create a ".env" file:

```bash
vi .env
```

with the following contents:

```text
export MODESHAPE_DB_PASSWORD=fcrepo
export LDAP_BIND_PASSWORD=... # See "FCRepo Directory LDAP AuthDN" in the "Identities" document on Box.
export JWT_SECRET=... # uuidgen | shasum -a256 | cut -d' ' -f1
```

ℹ️ **Note:** The "MODESHAPE_DB_PASSWORD" and "JWT_SECRET" are arbitrary, and
so can be different from the above, if desired. The only requirement for
"JWT_SECRET" is that it be "sufficiently long", which is accomplished by
the uuidgen command (but any "sufficiently long" string will work).

1) Deploy the stack:

```bash
source .env && docker stack deploy --with-registry-auth -c umd-fcrepo.yml umd-fcrepo
```

ℹ️ **Note:** The "--with-registry-auth" flag overrides any "latest" images in
the local Docker cache with the "latest" image from the Nexus -- even if the
image on the Nexus is older than the image in the local cache.

1) Switch back to the base "umd-fcrepo" directory:

```bash
cd ..
```

1) See the "umd-fcrepo URLs" section below for the list of accessible URLs.

## Starting from Scratch

In the following steps, the Docker images are built on the local workstation,
instead of being retrieved from the Nexus.

These steps should be used when testing custom versions of the images that
are not available from the Nexus. See the ["Quick Start"](#quick-start) section
above if you just want to use Docker images from the Nexus.

### Step 1: Clone the "umd-fcrepo" repository

1.1) Clone the "umd-fcrepo" repository and its submodules:

```bash
git clone --recurse-submodules git@github.com:umd-lib/umd-fcrepo.git
```

1.2) Switch into the "umd-fcrepo" directory:

```bash
cd umd-fcrepo
```

The "umd-fcrepo" directory will be considered the "base directory" for the
following steps.

### Step 2: (Optional) Build the "umd-camel-processors" jar

This step is only needed if a "SNAPSHOT" version of the "umd-camel-processors"
is being used, or if the tagged version is not in the Nexus:

2.1) Build and deploy the "umd-camel-processors" jar to the
Maven Nexus. This step is only needed if a "SNAPSHOT" version of the
"umd-camel-processors" is being used, or if the tagged version is not
in the Nexus:

```bash
cd umd-camel-processors
mvn clean deploy
cd ..
```

ℹ️ **Note:** Deploying to the Nexus means that the "umd-camel-processors"
version will be available to all users.

### Step 3: Deploy umd-fcrepo-docker stack

3.1) Switch to the base directory.

3.2) Switch to "umd-fcrepo-webapp" and build the Docker images for the
"umd-fcrepo" stack (see the umd-fcrepo-docker [README][umd-fcrepo-docker-README]
for canonical instructions):

```bash
cd umd-fcrepo-webapp
docker build -t docker.lib.umd.edu/fcrepo-webapp .

cd ../umd-fcrepo-messaging
docker build -t docker.lib.umd.edu/fcrepo-messaging .

cd ../umd-fcrepo-solr
docker build -t docker.lib.umd.edu/fcrepo-solr-fedora4 .

cd ../umd-fcrepo-docker

docker build -t docker.lib.umd.edu/fcrepo-fuseki fuseki
docker build -t docker.lib.umd.edu/fcrepo-fixity fixity
docker build -t docker.lib.umd.edu/fcrepo-mail mail
```

3.3) Create a ".env" file:

```bash
vi .env
```

with the following contents:

```text
export MODESHAPE_DB_PASSWORD=fcrepo
export LDAP_BIND_PASSWORD=... # See "FCRepo Directory LDAP AuthDN" in the "Identities" document on Box.
export JWT_SECRET=... # uuidgen | shasum -a256 | cut -d' ' -f1
```

ℹ️ **Note:** The "MODESHAPE_DB_PASSWORD" and "JWT_SECRET" are arbitrary, and
so can be different from the above, if desired. The only requirement for
"JWT_SECRET" is that it be "sufficiently long", which is accomplished by
the uuidgen command (but any "sufficiently long" string will work).

3.4) Deploy the stack:

```bash
source .env && docker stack deploy -c umd-fcrepo.yml umd-fcrepo
```

3.5) Switch back to the base directory:

```bash
cd ..
```

3.6) See the "umd-fcrepo URLs" section below for the list of accessible URLs.

## umd-fcrepo URLs

Once the containers start, the following URLs should be accessible:

* ActiveMQ admin console: <http://fcrepo-local:8161/admin>
* Solr admin console: <http://fcrepo-local:8983/solr/#/>
* Fuseki admin console: <http://fcrepo-local:3030/>
* Fedora repository REST API: <http://fcrepo-local:8080/fcrepo/rest/>
* Fedora repository login/user profile page: <http://fcrepo-local:8080/fcrepo/user/>

## Further Setup

At this point, the "umd-fcrepo" stack is up and running. Setting up the Plastron
and Archelon applications can take one of two paths, depending on developer
needs:

* Local Plastron and Archelon applications - Sets up instances of the "plastron"
and "archelon" applications running on the local workstation. This setup is
best when doing actual development on these applications.

See [plastron-archelon-local-setup.md](plastron-archelon-local-setup.md)

* Docker-based Plastron and Archelon applications - Sets up "plastron" and
"archelon" Docker stacks. This setup is best for testing, as it is most
similar to the Kubernetes enviroments.

See [plastron-archelon-docker-setup.md](plastron-archelon-docker-setup.md)

----
[umd-fcrepo-docker-README]: https://github.com/umd-lib/umd-fcrepo-docker/blob/develop/README.md#quick-star
