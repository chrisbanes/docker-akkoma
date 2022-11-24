# Akkoma

[Akkoma](https://akkoma.social/) is a federated social networking platform, compatible with GNU social and other OStatus implementations.

It actually consists of two components: a backend, named simply Akkoma, and various frontends which can be installed. It's main advantages are its lightness and speed.

This setup is heavily based on [angristan/docker-pleroma](https://github.com/angristan/docker-pleroma), with tweaks and changes needed to use Akkoma instead.

## Features

- Based on the elixir:alpine image
- Ran as an unprivileged user
- It works great

Sadly, this is not a reusable (e.g. I can't upload it to the Docker Hub), because for now Akkoma needs to compile the configuration. Thus you will need to build the image yourself, but I explain how to do it below.

## Build-time variables

- **`AKKOMA_VER`** : Akkoma version (latest commit of the [`stable` branch](https://akkoma.dev/AkkomaGang/akkoma) by default)
- **`GID`**: group id (default: `1000`)
- **`UID`**: user id (default: `1000`)

## Usage

### Installation

Create a folder for your Akoma instance. Inside, you should have the `build` folder, `docker-compose.yml` and `db.env` from this repo.

#### 1. Create folders

Create the various folders needed, and set ownership of the folders. The values should match the `UID` and `GID` used to build the image. Feel free to change the location of these folders, just remember to update the `docker-compose.yml` file too.

``` sh
mkdir uploads config static
chown -R 1000:1000 uploads config static
```

#### 2. Enable citext extension

Akkoma uses the `citext` PostgreSQL extension, here is how to add it:

```sh
docker-compose up -d db
docker exec -i akkoma_db psql -U akkoma -c "CREATE EXTENSION IF NOT EXISTS citext;"
docker-compose down
```

#### 3. (Optional) Configure Akkoma

Optionally configure Akkoma, see [Config Override](#config-override).

#### 4. Build and launch!

You can now build the image:

``` sh
docker-compose build
```

You can now launch your instance:

```sh
docker-compose up -d
```

The initial creation of the database schema will be done automatically. Check if everything went well with:

```sh
docker logs -f akkoma_web
```

#### 5. Create admin user

Make a new admin user using docker exec (replace `fakeadmin` with any username you like):

``` sh
docker exec -it akkoma_web mix pleroma.user new fakeadmin admin@domain.net --admin
```

At this point if you connect to Akkoma, you should hopefully see a page which tells you to install a frontend.

You can now setup a reverse proxy in a container or on your host by using the [example Nginx config](https://git.pleroma.social/pleroma/pleroma/blob/develop/installation/pleroma.nginx).

#### 6. Install frontend(s)

Akkoma does not include any frontends (web interfaces) by default, and instead provides easy methods to install them.

You can start off by installing the normal `pleroma-fe` by running:

``` sh
docker exec -it akkoma_web mix pleroma.frontend install pleroma-fe --ref stable
```

For more information on how Akkoma handles frontends, see the [Frontend Management](https://docs.akkoma.dev/stable/configuration/frontend_management/) documentation.

### Updating

By default, the Dockerfile will be built from the latest commit of the `stable` branch.

Thus to update, just rebuild your image and recreate your containers:

```sh
docker-compose pull # update the PostgreSQL if needed
docker-compose build .
docker-compose run --rm web mix ecto.migrate # migrate the database if needed
docker-compose up -d # recreate the containers if needed
```

If you want to run a specific commit, you can use the `AKKOMA_VER` variable:

```sh
docker build -t pleroma . --build-arg AKKOMA_VER=develop # a branch
docker build -t pleroma . --build-arg AKKOMA_VER=a9203ab3 # a commit
docker build -t pleroma . --build-arg AKKOMA_VER=v3.4.0 # a version
```

This value can also be set through `docker-compose.yml` as seen in the example file provided in this repository.

## Config

### Config override

For those that want to change and tweak Akkoma's any configuration properties which are are not exposed through environment variables, there is the option to provide your own `/config/config.exs`. This `config.exs` is read from wherever you mounted `/config` in your `docker-compose.yml` file. The config will be automatically read by the built-in config setup, and any values you set in this file will override the default setup.

### Secrets

By default, the keys and secrets needed for encryption will be generated automatically on first run, and stored in the `/config/secret.exs` file. There's nothing you need to do here, but if you wish to customize the values you can modify this file as required. You may also want to limit file permissions of this file.

=======

## Other Docker images

Here are other Pleroma Docker images that helped me build mine:

- [angristan/docker-pleroma](https://github.com/angristan/docker-pleroma)
- [potproject/docker-pleroma](https://github.com/potproject/docker-pleroma)
- [rysiek/docker-pleroma](https://git.pleroma.social/rysiek/docker-pleroma)
- [RX14/iscute.moe](https://github.com/RX14/kurisu.rx14.co.uk/blob/master/services/iscute.moe/pleroma/Dockerfile)
