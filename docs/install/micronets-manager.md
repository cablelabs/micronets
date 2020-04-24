## Installation guide for the Micronets Manager

### for NCCoE Build 3

Prerequisites:

* A Ubuntu 18.04 LTS server reachable by the server hosting the Micronet Manager instances
and any Micronets gateways
* Docker (v18.06 or higher)
* Docker-compose (v1.23.1 or higher)
* OpenSSL (1.0.2g or higher)
* curl
* nginx (1.14.0 or higher)

Instructions:

0. Install a newer version of docker-compose, if necessary. (Ubuntu 18.04 comes with an older version)

   Check the current version with:
   ```
   docker-compose --version
   ```
   
   If the version is earlier than v1.23.1 run the following to install a new version in `/usr/local/bin`:

   ```
   curl -L -O https://github.com/docker/compose/releases/download/1.24.1/docker-compose-Linux-`uname -m`
   sudo install -v -o root -m 755 docker-compose-Linux-`uname -m` /usr/local/bin/docker-compose
   chmod +x /usr/local/bin/docker-compose
   ```

0. Download the management script:

   ```
   curl -O https://raw.githubusercontent.com/cablelabs/micronets-manager/nccoe-build-3/scripts/mm-container
   curl -O https://raw.githubusercontent.com/cablelabs/micronets-manager/nccoe-build-3/scripts/docker-compose.yml
   sudo install -v -o root -m 755 -D -t /etc/micronets/micronets-manager.d mm-container
   sudo install -v -o root -m 644 -D -t /etc/micronets/micronets-manager.d docker-compose.yml
   ```

    Note: These instructions assume the default values contained in the mm-container management script.
    These values can be modified directly in your copy of the management script or overridden via command-line
    parameters.

0. Copy the mm server cert/key and the WS Proxy root CA cert created in earlier steps
   for use by the Micronets Manager docker container(s):

   ```
   sudo install -v -o root -m 600 -D -t /etc/micronets/micronets-manager.d/lib micronets-manager.{cert,key}.pem micronets-ws-root.cert.pem
   sudo touch /etc/micronets/micronets-manager.d/lib/micronets-ws-proxy.pkeycert.pem
   ```

   Note: Creating the `micronets-ws-proxy.pkeycert.pem` file is a work-around for an issue
   that will be fixed in the near future.

0. Generate a shared secret for communicating between the MSO Portal and the MM:

   ```
   sudo /etc/micronets/micronets-manager.d/mm-container create-mso-secret
   ```

    Note: This may be changed to generate the shared secret during the MSO Portal
          installation.

0. Download the Micronets Manager docker image:

   ```
   /etc/micronets/micronets-manager.d/mm-container pull
   ```

    Note: If you cannot connect to the Docker service, use "sudo usermod -aG docker <username>" to
          add the user account to the docker group.

0. Configure nginx for Micronets Manager:

   ```
   sudo /etc/micronets/micronets-manager.d/mm-container setup-web-proxy
   ```

   This will setup the folder to dynamically create forwarding entries for
   Micronets Manager instances as they're created/removed. But the site files in 
   `/etc/nginx/sites-available/` need to have the following added to the `server` 
   blocks to enable the forwarding of subscriber operations to the correct 
   Docker container.
   
   ```
   include /etc/nginx/micronets-subscriber-forwards/*.conf;
   ```

   For example:
   
   ```
   server {
        listen 443 ssl;
        listen [::]:443 ssl;

        root /var/www/mm.mydomain.in/html;

        index index.html index.htm index.nginx-debian.html;

        server_name mm.mydomain.in;

        location / {
                proxy_pass      http://localhost:8080/;
        }

        ssl_certificate /etc/letsencrypt/live/mm.mydomain.in/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/mm.mydomain.in/privkey.pem; # managed by Certbot

        include /etc/nginx/micronets-subscriber-forwards/*.conf;
   }
   ```

   Note that it's not necessary to restart nginx after these changes 
   since the `mm-container` script will cause nginx to reload files.
   But if you want to ensure your edits are OK, you can run 
   `sudo nginx -s reload` to have nginx reload the configuration files.

0. Start a Micronets Manager for a subscriber:

   ```
   /etc/micronets/micronets-manager.d/mm-container start <subscriber-name>
   ```

0. Verify the Micronets Manager is running:

   ```
   /etc/micronets/micronets-manager.d/mm-container logs <subscriber-name>
   ```

   - You should see output like the following:

    ```
    2020-03-31T10:52:22.771430740Z Debugger listening on ws://0.0.0.0:9229/4da874cb-3442-4d90-9b6e-d05f9defe91b
    2020-03-31T10:52:22.771503633Z For help see https://nodejs.org/en/docs/inspector
    2020-02-26 20:35:14 ESC[32minfoESC[39m [index.js]: MONGODB URI : "mongodb://mongodb/micronets"
    2020-02-26 20:35:14 ESC[32minfoESC[39m [index.js]: MONGOOSE URI : "mongodb://mongodb/micronets"
    ...
    2020-02-26 20:35:14 ESC[32minfoESC[39m [index.js]: Feathers application started on "http://0.0.0.0:3030"
    2020-02-26 20:35:14 ESC[32minfoESC[39m [index.js]: Public base URL: "https://dev.mm-api.micronets.in/sub/auntbetty/api"
    ```

