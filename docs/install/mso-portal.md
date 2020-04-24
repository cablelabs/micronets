## Installation guide for Micronets MSO Portal

### for NCCOE Build 3

Prerequisites:

* A server reachable by the server hosting the Micronet Manager instances
   and whatever device is running the mobile application.
* Docker (v18.06 or higher)
* Docker-compose (v1.23.1 or higher)
* OpenSSL (1.0.2g or higher)
* nginx and requisite certificates if https is to be supported


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

0. Download the MSO Portal management script:

   ```
   curl -O https://raw.githubusercontent.com/cablelabs/micronets-mso-portal/nccoe-build-3/scripts/mso-portal
   curl -O https://raw.githubusercontent.com/cablelabs/micronets-mso-portal/nccoe-build-3/scripts/docker-compose.yml
   sudo install -v -o root -m 755 -D -t /etc/micronets/mso-portal.d mso-portal
   sudo install -v -o root -m 644 -D -t /etc/micronets/mso-portal.d docker-compose.yml
   ```

    Note: The mso-portal management script contains default values can be modified directly in your 
    copy of the management script or overridden via command-line parameters. Run 
    `/etc/micronets/mso-portal.d --help` to see the options.

0. Download the MSO Portal docker image:

   ```
   /etc/micronets/mso-portal.d/mso-portal docker-pull
   ```

   Note: If you cannot connect to the Docker service, you can use 
         `sudo usermod -aG docker <username>` to add the user account to the docker group.

0. Generate a shared secret for enabling communication between Micronet Manager instances and the MSO Portal:

   ```
   sudo /etc/micronets/mso-portal.d/mso-portal create-mso-secret
   ```

   Note: This value will need to be copied to the Micronets Manager host server to 
         allow MM instances to access the MSO Portal APIs.

0. Configure MSO Portal URLs:
 
   Modify `/etc/micronets/mso-portal.d/mso-portal` to refer to the base URI for the MSO API
   and the base URL for the websocket proxy. 

   Set the `DEF_MSO_API_BASE_URL` variable to the base URI that the MSO can be reached at by 
   the Micronets Manager instances. And set the `DEF_WS_PROXY_BASE_URL` variable to the base 
   URI for creating endpoints on the websocket proxy where Micronet gateways and their peer 
   Micronet Managers can connect to each other. 
   
   Note that the makeup of these URLs will depend on where you have the MSO Portal 
   installed and how the frontend web service is configured. (some details and exmaple with 
   nginx below)

   e.g.
   ```
   DEF_MSO_API_BASE_URL="https://dev.mso-portal-api.micronets.in:443"
   DEF_WS_PROXY_BASE_URL="wss://ws-proxy-api.micronets.in:5050/micronets/v1/ws-proxy/gw"
   ```
   
0. Start the MSO Portal docker image:

   ```
   sudo /etc/micronets/mso-portal.d/mso-portal docker-run
   ```

0. Verify the MSO Portal started up successfully:

   ```
   /etc/micronets/mso-portal.d/mso-portal logs
   ```

   - You should see output like the following at the end of the log:

    ```
    Feathers application started on "http://0.0.0.0:3210"
    Feathers  webSocketBaseUrl "wss://ws-proxy-api.micronets.in:5050/micronets/v1/ws-proxy/gw"
    Feathers  publicApiBaseUrl "https://dev.mso-portal-api.micronets.in:443"
    ```

0. To securely expose the MSO API, configure your https proxy to redirect to localhost port
   3210. For example, an enginx configuration in `/etc/nginx/sites-enabled/micronets-mso-portal` 
   managed by Let's Encrypt could contain:
   
   ```
   # Micronets MSO Portal API
   server {
        # SSL configuration
        
        listen 443 ssl;
        listen [::]:443 ssl;

        server_name mso-portal-api.my-domain.org;

        location / {
                proxy_pass      http://localhost:3210/;
        }
        ssl_certificate /etc/letsencrypt/live/mso-portal-api.my-domain.org/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/ mso-portal-api.my-domain.org/privkey.pem; # managed by Certbot
   }
```

