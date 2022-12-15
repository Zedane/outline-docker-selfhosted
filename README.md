# Outlinewiki completely self-hosted on docker

TLDR; Follow the quick setup, if you want to get outline running in an enclosed environment. It will be an IP-based setup, meaning you will be able to access outline via your machines IP. The custom setup includes setting up a reverse proxy.

<br>

For completely self-host outline we'll use
- [Keycloak](https://www.keycloak.org/) for authentication
- [Minio](https://hub.docker.com/r/minio/minio/) for storage
- [Postgres](https://hub.docker.com/_/postgres) and [Redis](https://hub.docker.com/_/redis)

[Outline](https://www.getoutline.com/) is quite difficult to set up and the [docs](https://app.getoutline.com/s/770a97da-13e5-401e-9f8a-37949c19f97e/doc/docker-7pfeLP5a8t) don't seem to work out of the box. So I referred to [this blogpost](https://blog.gurucomputing.com.au/doing-more-with-docker/deploying-outline-wiki/) when I set up my own instance. 

## Prequisites

> If you already have a running docker environment you can skip this section.

First you need to install a few dependencies.

```shell
sudo apt-get update
```

```shell
sudo apt-get install ca-certificates curl gnupg lsb-release nano wget
```

Then install docker. You can refer to the [docker documentation](https://docs.docker.com/engine/install/ubuntu/) for more information.

Set up the repository:

```shell
sudo mkdir -p /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Install the engine:

```shell
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

To verify your docker installation run:

```shell
sudo docker run hello-world
```


## Quick setup

> ### *Use this method if you just want to try out outline. This method is not safe for production environments.*

The quick setup was originally done on ubuntu 22.04, but should work on other destributions as well. Make sure, that you have the following dependencies already installed:

### 1. Configure Authentication

  **1.1 Run a Keycloak container**
  
  We'll use the [Keycloak quick setup](https://www.keycloak.org/getting-started/getting-started-docker) to run a keycloak-instance as our authentication provider.
  ```shell
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

### 3. Get the `docker-compose.yml`. 

The quick setup has most variables already prefilled.
> **Do NOT use this docker-compose.yml in a production environment!**

```shell
wget https://github.com/zedane/outline-docker-selfhosted/
```

### 4. Create a `.env` file

```shell
nano .env
```

and paste the contents into the file. Save the file when you're done.

```shell
# .env
HOST_IP=<IP-address here>
OIDC_SECRET_KEY= # From step 1.5
OL_SECRET= # use openssl rand -hex 32
CLEINT_SECRET= # use openssl rand -hex 32
```

### 5. Run the containers

You can pass `-d` as parameter, if you want the containers to run in the background.

```shell
sudo docker-compose up
```

Congratulations! Now you can open up `http://<your servers IP>:8080/` and use outline. In the custom setup you'll se how to set up a reverse proxy an production-ready authentication, so if you're interested, check out that setup. 

## Custom setup

The setup uses example domains. When following the instructions, replace `example.com` with your own domain.

### 1. Reverse Proxy

  **1.1 Load the docker-compose.yml**
  ```shell
  wget -O proxy/docker-compose.yml https://github.com/Zedane/outline-docker-selfhosted/custom/proxy/docker-compose.yml
  ```

  **1.2 Create a .env file**

  ```shell
  nano proxy/.env
  ```

  ```shell 
  # .env
  MYSQL_PASSWORD=
  MYSQL_ROOT_PASSWORD=
  ```

  **1.3 Build the containers**
  ```shell
  sudo docker-compose -f proxy/docker-compose.yml up -d
  ```
  **1.4 Admin panel settings**
  Open your browser and navigate to the nginx admin panel. Use your servers IP address to do so, e.g. `http://192.168.1.1:81/`, and log in using the default credentials 'admin@example.com' and 'changeme'. You'll be asked to change the credentials when you first log in. Then import an SSL-certificate or request one from *Let's Encrypt*. You could use a wildcard certificate, although it's not recommended to do so.

  Then add these 4 *Proxy Hosts*:

  | Domain Names | Scheme | Forwarded Hostname / IP | Forwarded Port | Websocket support | Block Common Exploits | Access list | SSL Certificate | Force SSL | HTTP/2 Support | HSTS Enabled |
  |-|-|-|-|-|-|-|-|-|-|-|
  | login.example.com | http | keycloak | 9090 | Yes | Yes | public | *select fitting* | Yes | Yes | No |
  | outlinewiki.example.com | http | outline | 443 | Yes | Yes | public | *select fitting* | Yes | Yes | Yes |
  | data.example.com | http | outline_minio | 9000 | Yes | Yes | public | *select fitting* | Yes | Yes | Yes |
  | data-admin.example.com | http | outline_minio | 9001 | Yes | Yes | public | *select fitting* | Yes | Yes | Yes |

### X. Keycloak authentication provider

```shell
# .env
POSTGRES_PASSWORD=
KEYCLOAK_PASSWORD=
```

### X. Outline setup

> Some values might not work through the env file. You might have to edit the docker-compose directly, though this method is not recommended!

```shell
# .env
POSTGRES_PASSWORD=
MINIO_ROOT_PASSWORD=
MINIO_BROWSER_REDIRECT_URL=https://data-admin.example.com
SECRET_KEY=
UTILS_SECRET=
HOST_URL=https://outlinewiki.example.com
DATA_URL=https://data.example.com
OIDC_CLIENT_SECRET=
KEYCLOAK_HOST=https://login.example.com
```