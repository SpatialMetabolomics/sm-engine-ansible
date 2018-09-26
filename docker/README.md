## Description

This directory contains a docker-compose configuration for setting up instances of the METASPACE
platform for personal and development use.

This configuration has not been secured for production use and should not be deployed to any server
with the intent of providing public access without first considering the security implications.
In particular, administrative ports for back-end services have not been closed in this
configuration, and easily guessable passwords are used and stored in plain text.

## Usage

### Install Docker

```
sudo apt-get install docker-compose git
sudo snap install docker
```

After installation, log out and log back in to start the Docker daemon. If this step isn't done,
docker will hang when trying to start containers.

### Docker config file

Copy `.env.example` to `.env` and customize it if needed.

> TODO: Change dockerfiles to accept a commit hash instead of a branch as an argument

### Self-contained installation

Run `docker-compose up -d` and jump to the "Import data" section

### Development installation

In a development setup, SM projects are run from volumes that are linked to the filesystem.
These projects are expected to be checked out in the directory defined by the `DEV_ROOT` variable
in the `.env` file, which defaults to `../..`, the directory above where this `sm-engine-ansible`
repository is checked out.
e.g. the `sm-webapp` container will run from code in `../../sm-webapp` relative to the
`docker-compose.yml` file.

After setting up the `.env` file, run `setup-dev-env.sh` to automatically clone the repositories
into `DEV_ROOT` and set up the rest of the development environment.

### Import data

```bash
docker-compose run --rm mol-db /install-dbs.sh
./fetch-mol-images.sh
```

### Configuration

If running in a development setup, all config files will need to be updated with the following
settings. It may help to copy the config files from the configuration directories that are
used for self-contained docker installations: `./mol-db/config`, `./sm-engine/conf`, etc.

* Elastic search host/user/pass: `elasticsearch` / `elastic` / `elastic`
* Postgresql host: `postgres`
    * Admin db/user/pass: `postgres` / `postgres` / `postgres`
    * Mol-db db/user/pass: `mol_db` / `mol_db` / `password`
    * SM db/user/pass: `sm` / `sm` / `password`
* RabbitMQ host/user/pass: `rabbitmq` / `rabbitmq` / `rabbitmq`
* Hosts for SM services: `mol-db`, `sm-api`, `sm-graphql`
* Data directories: `/opt/data`

> *TODO:* Pull configuration templates from their respective repositories
> and use a Jinja-based tool to populate them with defaults so that the configuration isn't
> hard-coded

### Accessing METASPACE

* http://localhost:8999/ - Main site

Development tools:

* http://localhost:5601/ - Kibana
* http://localhost:9000/ - Adminer database management tool. Use "postgres" for System, Server,
    Username, Password and "sm" for Database
* http://localhost:15672/ - RabbitMQ management interface

Watching application logs:

* `docker-compose logs --tail 5 -f mol-db sm-api sm-daemon sm-graphql sm-webapp`

Rebuilding the Elasticsearch index:

* `docker-compose run --rm sm-api /rebuild-es-index.sh`
