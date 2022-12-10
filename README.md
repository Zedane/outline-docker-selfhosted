# Outlinewiki completely self-hosted on docker

TLDR; Follow the quick setup, if you want to get outline running in an enclosed environment. It will be an IP-based setup, meaning you will be able to access outline via your machines IP. The custom setup includes setting up a reverse proxy.

<br>

For completely self-host outline we'll use
- [Keycloak](https://www.keycloak.org/) for authentication
- [Minio](https://hub.docker.com/r/minio/minio/) for storage
- [Postgres](https://hub.docker.com/_/postgres) and [Redis](https://hub.docker.com/_/redis)

[Outline](https://www.getoutline.com/) is quite difficult to set up and the [docs](https://app.getoutline.com/s/770a97da-13e5-401e-9f8a-37949c19f97e/doc/docker-7pfeLP5a8t) don't seem to work out of the box. So I referred to [this blogpost](https://blog.gurucomputing.com.au/doing-more-with-docker/deploying-outline-wiki/) when I set up my own instance. 

## Quick setup

> Use this method if you just want to try out outline. This method is not safe for production environments.

The quick setup was originally done on ubuntu 22.04, but should work on other destributions as well. Make sure, that you have the following dependencies already installed:

- `docker` 
- `docker-compose` 
- `wget`
- `nano`

### 1. Configure Authentication

  **1.1 Run a Keycloak container**
  
  We'll use the [Keycloak quick setup](https://www.keycloak.org/getting-started/getting-started-docker) to run a keycloak-instance as our authentication provider.
  ```
  docker run -p 9090:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:20.0.1 start-dev
  ```
  We'll expose the container on `Port 9090`, because we'll use 8080 later.

  **1.2 Open the Admin console**
  
  Now open up your browser and go to `http://<your servers IP>:9090/admin` and use `admin` and password `admin` to log in.

  **1.3 Create a client**
  
  You'll need to create a new client and set the following values:
  - ClientId: outline
  - 

  **1.4 Create a user**

  **1.5 Check the Secret key**

  You'll need this secret key in step 3.

### 2. Get the `docker-compose.yml`. 
The quick setup has most variables already prefilled.
> **Do NOT use this docker-compose.yml in a production environment!**

```
wget https://github.com/zedane/outline-docker-selfhosted/
```

### 3. Create a `.env` file

```
nano .env
```

and paste the contents into the file. Save the file when you're done.

``` .env
HOST_IP=<IP-address here>
OIDC_SECRET_KEY= # From step 1.5
OL_SECRET= # use openssl rand -hex 32
CLEINT_SECRET= # use openssl rand -hex 32
```

### 4. Run the containers

You can pass `-d` as parameter, if you want the containers to run in the background.

```
sudo docker-compose up
```

Congratulations! Now you can open up `http://<your servers IP>:8080/` and use outline. In the custom setup you'll se how to set up a reverse proxy an production-ready authentication, so if you're interested, check out that setup. 

## Custom setup
