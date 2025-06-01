
# Table of Contents
- [Install Nextcloud on a Docker Compose Stack](#installing-nextcloud)
- [Configure Nextcloud](#configure-nextcloud)
- [Troubleshooting Nextcloud Issues](#troubleshooting-nextcloud)
  - [Connection to database or SQLDB failed to authorize the admin user](#connection-to-database-issue)
  - [Incorrect username or password while logging in](#incorrect-username-or-password)
  - [Nextcloud does not have permission to write to the apps directory](#nextcloud-apps-permission-issue)
  - [Not a trusted domain](#nextcloud-trusted-domain-issue)
- [Setup Collabora in Nextcloud for editing documents](#setup-collabora)
- [Troubleshooting Collabora Issues](#troubleshooting-collabora)
  - [Collabora shows socket error when trying to edit a document](#collabora-socket-error)
- [Setup Next Cloud Client on your devices](#setup-next-cloud-client-on-your-devices)
    - [on iOS](#on-ios)



# Install Nextcloud on a Docker Compose Stack <a name="installing-nextcloud"></a>

_NOTE:_ Nextcloud does not contain a solid docker compose file that lets you install only nextcloud (not the AIO) along with the data databases and related services. The below docker compose files is a modified version based on online documenatations, along with some issues that I faced while installing Nextcloud.

_NOTE:_ Instead of Nextcloud, you can use the AIO version of Nextcloud, which is available as a docker image.

1. Go to `Portainer` --> `local` environment --> `dashboard` --> `stack` --> `Add Stack`
2. Provide a name, say `Nextcloud`, choose the `Web Editor`, and paste the below yaml into the editor.

```yaml
---

services:
  nextcloud:
    image: nextcloud:latest
    container_name: nextcloud
    restart: unless-stopped
    depends_on:
      - nextclouddb
      - redis
    ports:
      - 9280:80
      - 9243:443
    volumes:
      - /mnt/somefolder/docker/nextcloud/html:/var/www/html
      - /mnt/somefolder/docker/nextcloud/html/customapps:/var/www/html/custom_apps
      - /mnt/somefolder/docker/nextcloud/html/config:/var/www/html/config
      - /mnt/somefolder/docker/nextcloud/html/data:/var/www/html/data
    environment:
      - PUID=${PUID}
      - PGID=${PGID}
      - TZ=${TZ}
      - MYSQL_DATABASE=${MYSQL_DATABASE}
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
      - MYSQL_HOST=${MYSQL_HOST}
      - REDIS_HOST=redis

  nextclouddb:
    image: mariadb
    container_name: nextcloud-db
    restart: unless-stopped
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
    volumes:
      - /mnt/somefolder/docker/nextcloud/nextclouddb:/var/lib/mysql
    ports:
      - 3306:3306
    environment:
      - PUID=${PUID}
      - PGID=${PGID}
      - TZ=${TZ}
      - MYSQL_RANDOM_ROOT_PASSWORD=true
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
      - MYSQL_DATABASE=${MYSQL_DATABASE}
      - MYSQL_USER=${MYSQL_USER}
      
  collabora:
    image: collabora/code
    container_name: nextcloud-collabora
    restart: unless-stopped
    environment:
      - PUID=${PUID}
      - PGID=${PGID}
      - TZ=${TZ}
      - password=${COLLABORA_PASSWORD}
      - username=${COLLABORA_USER}
      - domain=${nextcloud_domain}
      - extra_params=--o:ssl.enable=false --o:ssl.termination=true
    ports:
      - 9980:9980

  redis:
    image: redis:alpine
    container_name: nextcloud-redis
    volumes:
      - /mnt/somefolder/docker/nextcloud/redis:/data    
```

3. Configure the environment variables under `Environment Variables` --> `Advanced Mode`.
4. Click on `Deploy the stack` which will deploy 4 containers: `nextcloud`, `nextclouddb`, `collabora`, and `redis` under the network `nextcloud_default`.
5. Once the stack is deployed, you can access Nextcloud at `http://<your-pi-ip>:9280`

# Configure Nextcloud <a name="configure-nextcloud"></a>
1. Open a web browser and go to `http://<your-pi-ip>:9280`.
2. You will see the Nextcloud setup page.
3. Fill in the details:
   - **Admin account**: Create an admin account by providing a username and password.
   - **Data folder**: Ensure the data folder points to `/var/www/html/data`.
   - **Choose a database**: Select `MySQL/MariaDB`.
   - **Database configuration**: 
     - Database user: `${MYSQL_USER}`
     - Database password: `${MYSQL_PASSWORD}`
     - Database name: `${MYSQL_DATABASE}`
     - Database host: `nextclouddb`
4. Click on `Finish setup`. This will takes few minutes as Nextcloud will set up the database and configure the application.
5. Once the setup is complete, you will be redirected to the Nextcloud dashboard.

_NOTE:_ I faced several issues mentioned, so please follow the below steps to resolve them if you face any issues.

# Troubleshooting Nextcloud Issues <a name="troubleshooting-nextcloud"></a>

## Connection to database or SQLDB failed to authorize the admin user <a name="connection-to-database-issue"></a>
If you encounter an error stating "Connection to database or SQLDB failed to authorize the admin user", it usually indicates that the database connection details are incorrect or the database service is not running properly.
1. **Check Database Service**: Ensure that the `nextclouddb` service is running.
2. **Verify Environment Variables**: Double-check the environment variables in your `docker-compose.yml` file to ensure that `${MYSQL_USER}`, `${MYSQL_PASSWORD}`, and `${MYSQL_DATABASE}` are set correctly, and the same is reflected in the Nextcloud setup page.

If this does not resolve the issue, you can try the following steps:
1. Clear the volumes for the `nextcloud` and `nextclouddb` services.
2. Remove the containers for `nextcloud`, `nextclouddb`, `collabora`, and `redis`.
3. Rerun the stack.

This has fixed the issue for me.

## Incorrect username or password while loggin in <a name="incorrect-username-or-password"></a>
If you encounter an error stating "Incorrect username or password" while trying to log in to Nextcloud, it may be due to the following reasons:
1. **Truely Incorrect Credentials**: Double-check the username and password you created during the Nextcloud setup.

If this does not resolve, it could be because of the cache.
1. Clear the cookies and cache in your web browser.
2. Open a private/incognito window in your browser and try logging in again with the same credentials.

This should resolve the issue.

## Nextcloud does not have permission to write to the apps directory <a name="nextcloud-apps-permission-issue"></a>
If you encounter an error stating "Nextcloud does not have permission to write to the apps directory", it usually indicates that the permissions for the Nextcloud data directory are not set correctly.
1. Got to the terminal and SSH into your Raspberry Pi.
2. Run the following command to set the correct permissions for the Nextcloud data directory:
   ```bash
   sudo chmod -R 755 /mnt/somefolder/docker/nextcloud/html/data
   ```
or anyother folder that has the issue. This fixed the issue for me.

## Not a trusted domain <a name="nextcloud-trusted-domain-issue"></a>
If you encounter an error stating "Not a trusted domain" when accessing Nextcloud, it means that the domain you are trying to access is not listed in the trusted domains configuration.

1. Open the `config.php` file located in `/mnt/somefolder/docker/nextcloud/html/config/`.
2. Add your domain or IP address to the `trusted_domains` array. For example:
   ```php
   'trusted_domains' =>
   array (
     0 => 'localhost',
     1 => 'your-pi-ip-address',
   )
   ```
3. Save the `config.php` file and restart the Nextcloud container.

This fixed the issue for me. After making this change, you should be able to access Nextcloud without any "Not a trusted domain" error.

# Setup Collabora in Nextcloud for editing documents <a name="setup-collabora"></a>

_NOTE:_ Make sure both `nextcloud` and `collabora` containers are running and they are configured via `cloudflare tunnel` or any other reverse proxy to access them via a domain name.

_NOTE:_ Add the `nextcloud domain name` to the trusted domains in the `config.php` file of Nextcloud as mentioned above.

1. Go to `Nextcloud` --> `Apps` --> `Discover`, and download and install the  `Nextcloud Office` app.
2. Go to `Nextcloud` --> `Administation Settings` --> `office` --> `Use your own server`, and enter the URL of the Collabora server, which is `https://<your-collabora-domain>`.
    - You will see a message saying "Collabora Online server is reachable" if the configuration is correct.
3. Now you can create or edit documents in Nextcloud using Collabora.

# Troubleshooting Collabora Issues <a name="troubleshooting-collabora"></a>

## Collabora show socket error when trying to edit a document <a name="collabora-socket-error"></a>

If you encounter a socket error when trying to edit a document in Nextcloud using Collabora, it usually indicates that the Collabora server is not reachable or not configured correctly.

1. **Check Collabora Service**: Ensure that the `nextcloud-collabora` service is running. See the logs of the `collabora` container by going to `Portainer` --> `local` environment --> `containers` --> `nextcloud-collabora` and clicking on `Logs`.
2. **Error 301**: If you see a message like `Issues with HTTP Status Moved (Permanently) HTTP 301 Nextcloud`, then it means that the collabora server is trying to reach the Nextcloud server using the HTTP protocol instead of HTTPS.

This is a *known issue* as of May 2025, and the fix is given here: [Issues with HTTP Status Moved (Permanently) HTTP 301 Nextcloud, Collabora & Nginx Reverse Proxies #11287
](https://github.com/CollaboraOnline/online/issues/11287)

I follower the steps mentioned in the link, by updating the `config.php` file to adding the following line:
```php
'overwriteprotocol' => 'https',
```
This fixed the issue for me and was able to edit documents in Nextcloud using Collabora.


# Setup Next Cloud Client on your devices

## on iOS
1. Download the Nextcloud app from the [App Store](https://apps.apple.com/us/app/nextcloud/id1125420102).
2. Open the app and enter your Nextcloud server URL (e.g., `https://your-nextcloud-domain`).
3. Log in with your Nextcloud username and password.
4. Once logged in, you can access your files, upload photos, and sync data between your iOS device and Nextcloud.