# Installation instructions

## Check requirements

### Ensure that docker is installed
To install the project, you need to have [Docker](https://www.docker.com/products/docker-desktop/) installed on your machine. 

You can download the installer from the official website or *use following command to install docker on your machine**.

```bash
command -v docker > /dev/null 2>&1 || \
(curl -fsSL https://get.docker.com | sudo sh - && \
sudo gpasswd -a $(whoami) docker && \
sudo reboot)
```

> Note: To start installation process you must get the **Github access token** from **Referral Factory**.

Create a folder and goto the folder.

```bash
mkdir project && cd project
```

---

## Installation script

To run the setup with the installation script, you have to download the script file.

```bash
curl -o setup https://referral-factory.com/fetch/encryption && \
chmod +x setup
```

To start the installation process, run the following command.

```bash
./setup
```

### Installation steps
After you run the installation script, you will be asked to enter the following information.

1. **Github access token** - You can get the token from Referral Factory.
 
1. **Application Domain** - The domain name of the application. (e.g. `mydomain.com`)

1. **Automatic updates** - If you want to have the automatically updated latest version of the application, you can set it to `y`. Otherwise, you can update it manually by setting it to `n`.
 
1. **Two-factor authentication** - If you want to enable two-factor authentication, you can set it to `y`. Otherwise, you can set it to `n`.

1. **Superuser email** - The email address of the superuser. You will use that email to login the dashboard.

1. **Referral factory API URL** - The URL of the Referral Factory API. You can leave it empty if you are not using Referral Factory for testing purposes.

1. **Use external database** - If you want to use an external database connection, you can set it to `y`. Otherwise, you can use mysql inside docker container by setting it to `n`.
    > Note: In case of using an external database, you need to provide additional information.

    1. **Database connection** - The type of the database connection. You can choose between `mysql`, `pgsql`, `sqlite`.

    1. **Database host** - The hostname or ip of the database server.

   1. **Database port** - The port of the database server.

   1. **Database name** - The name of the database.

   1. **Database username** - The username of the database.

1. **Database password** - The password for the database connection. In case of using an external database, you need to provide the password for the database connection. In case of using mysql inside docker container, you can provide any password. 

1. **Application forwading port** - The port on which the application will be available. You can set any port number (e.g. `8080`).

1. **Database forwarding port** - The port on which the database will be available. You can set any port number (e.g. `3306`). 
    > This step is only available if you are using mysql inside docker container.
   
After finishing the installation process, application will start.

### Next steps
If you need to update the application to the latest version manually, you can run a following command.

```bash
./setup switch old_verion_number new_version_number
```

---

## Manual installation

### Prepare configuration 

Create a new file with name `.env.app` with following content.

```bash
DOMAIN=mydomain.com
GITHUB_ACCESS_TOKEN=your_token
REPO_AUTO_UPDATE=false
OTP_ENABLED=false
SUPERUSER_EMAIL=admin@example.com
DB_PASSWORD=my_supper_secret_password
USE_EXTERNAL_DB=false
```

Set `REPO_AUTO_UPDATE` to `true` if you want to have the automatically updated latest version of the application. Set to `false` if you want to update it manually.

Set `OTP_ENABLED` to `true` if you want to enable two-factor authentication.

`SUPERUSER_EMAIL` is the email address of the user which will be used for login to application. Password for the first login will be `password`.

`DB_PASSWORD` is the password for the database connection.

Set `USE_EXTERNAL_DB` to `true` if you want to use an external database connection. Set to `false` if you want to use mysql inside docker container.

In case of using an external database, you need to set the following environment variables in `.env.app` file.

```bash
DB_CONNECTION=mysql
DB_HOST=your_db_host
DB_PORT=your_db_port
DB_DATABASE=your_db_name
DB_USERNAME=your_db_user
```

Available values for database connection are `mysql`, `pgsql`, `sqlite`.

`DB_HOST` is the hostname or ip of the database server.

`DB_PORT` is the port of the database server.

`DB_DATABASE` is the name of the database.

`DB_USERNAME` is the username of the database.

### Run the docker container

Lets say the current release number is `1.0.14`.
We will use that number in the following commands.

**For internal database connection, run the following command.**

```bash
APP_RELEASE=1.0.14 && \
FORWARD_APP_PORT=8080 && \
FORWARD_MYSQL_PORT=3306 && \
docker run -d \
       -p $FORWARD_APP_PORT:8080 \
       -p $FORWARD_MYSQL_PORT:3306 \
       -v ./.env.app:/var/www/html/.env.app \
       -v encryption-redis:/var/lib/redis \
       -v encryption-mysql:/var/lib/mysql \
       -v ./logs:/var/www/html/storage/logs \
       --restart=unless-stopped \
       --name encryption-$APP_RELEASE referralfactory/encryption-app:$APP_RELEASE
```

**For external database connection, run the following command.**

```bash
APP_RELEASE=1.0.14 && \
FORWARD_APP_PORT=8080 && \
docker run -d \
       -p $FORWARD_APP_PORT:8080 \
       -v ./.env.app:/var/www/html/.env.app \
       -v encryption-redis:/var/lib/redis \
       -v ./logs:/var/www/html/storage/logs \
       --restart=unless-stopped \
       --name encryption-$APP_RELEASE referralfactory/encryption-app:$APP_RELEASE
```

You can replace `APP_RELEASE` with the new release number.
I above examples we used `FORWARD_APP_PORT` and `FORWARD_MYSQL_PORT` as forwarding ports for the application and mysql database.
You can put any port number you want.

### Switch to the new release manually

First of all you need to check latest available release of the application on settings page of your application by visiting [https://yourdomain.com/settings](https://yourdomain.com/settings).

On `App Version Information` section you can see if you have a new available release.

> Note: For `Automatic updates` you can set `REPO_AUTO_UPDATE` to `true` in `.env.app` file. In that case this section will always show that app is up to date.

If you see that there is an available release, you can do the following steps.

<br>

1. You must copy `storage` folder from the current running docker container and store it on a host machine, to use it later in the new docker container.
Run the following command.

```bash
APP_RELEASE=1.0.14 && \
docker cp encryption-$APP_RELEASE:/var/www/html/storage ./storage
```
<br>

2. Stop the current running docker container.
```bash
APP_RELEASE=1.0.14 && \
docker stop encryption-$APP_RELEASE
```
<br>

3. Run the new docker container with the new release.

> Note: Lets say you have a new release 1.0.15.
> Just replace the `APP_RELEASE` with `1.0.15` and run the command for starting application described in the previous section.

<br>

4. Copy the `storage` folder from the host machine to the new docker container.

```bash
APP_RELEASE=1.0.15 && \
docker cp ./storage encryption-$APP_RELEASE:/var/www/html/ && \
docker exec -u root encryption-$APP_RELEASE chown -R encryption:encryption /var/www/html/storage && \
rm -rf ./storage
````

<br>

5. Remove the old docker container.

If new release is running correctly, you can remove the old container.
```bash
APP_RELEASE=1.0.14 && \
docker rm encryption-$APP_RELEASE
```

---

## Nginx as a reverse proxy

You can access the application by visiting [http://yourdomain.com:APP_PORT](http://yourdomain.com:8080).

Also you can setup `Nginx` as a reverse proxy to access the application on port `80` or `443` to have secure connection.

Simple `Nginx` configuration for reverse proxy to access on port 443 with ssl.

```nginx
server {
    listen  80;
    server_name yourdomain.com;

    return  301 https://$host:443$request_uri;
}

server {
    listen 443 ssl;
    server_name yourdomain.com;

    ssl_certificate         /etc/nginx/certs/certificate.pem;
    ssl_certificate_key     /etc/nginx/certs/certificate.key;

    location / {
        proxy_set_header    X-Real-IP $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header    Host $host;
        proxy_set_header    X-Forwarded-Proto $scheme;
        proxy_set_header    X-Forwarded-Host $host;
        proxy_set_header    X-Forwarded-Port $server_port;
        proxy_pass          http://APP_IP:APP_PORT;
    }
}
```

### Additional Notes:

Replace `APP_IP` with the IP address of the server where the application is running. You can choose `localhost` if you are running the application on the same machine.

Replace `yourdomain.com` with your domain name and `APP_PORT` with the port number on which the application is running.

Replace `APP_PORT` with the port number on which the application is running.

Also you have to care about ssl certificate generation. You can use [Let's Encrypt](https://letsencrypt.org/) to generate ssl certificate for your domain.

---

## Automatic updates notes

If you want to have the application automatically updated to the latest version, you can set `REPO_AUTO_UPDATE` to `true` in `.env.app` file.

Also you have to setup the following cron job on your server.

```bash
# create file for logging cron job
touch $HOME/autoupdates.log && \
echo "*/5 * * * * curl -fsSL https://referral-factory.com/fetch/encryption | bash -s -- update  >> $HOME/autoupdates.log" | crontab -
```
