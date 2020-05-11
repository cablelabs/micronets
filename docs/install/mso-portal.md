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
 
   Some settings in the `/etc/micronets/mso-portal.d/mso-portal` file needs to be 
   modified to reflect the "public" endpoints of some Micronets services.
    
   The value of the `DEF_MSO_API_BASE_URL` variable will depend on where you 
   have the MSO Portal installed and how the frontend web service is configured.
   It must contain a base URI that Micronet Manager instances can use to reach the 
   MSO API. 

   e.g. If the nginx path to the MSO Portal web services is setup using the following 
   `location` entry in the nginx `server` block for the webservice:
   ```
    location /micronets/mso-portal/ {
        proxy_pass http://127.0.0.1:3210/;
    }
   ```

   Would require the `DEF_MSO_API_BASE_URL` path variable to be set to:

   ```
   DEF_MSO_API_BASE_URL="https://my-server.org/micronets/mso-portal/"
   ```
   
   The value of the `DEF_WS_PROXY_BASE_URL` variable only has to be updated to reflect
   the host and port where the Websocket Proxy is installed. The path prefix must reflect
   the URL Prefix configured in the websocket proxy - by default `/micronets/v1/ws-proxy/`.
   For example:
   
   ```
   DEF_WS_PROXY_BASE_URL="wss://my-server.org:5050/micronets/v1/ws-proxy/gw"
   ```

   Note: This variable should match the `MM_GATEWAY_WEBSOCKET_BASE_URL` setting 
         in the `/etc/micronets/micronets-manager.d/docker-compose.yml` file
         on the server hosting the Micronets Manager containers.

0. Start the MSO Portal docker image:

   ```
   sudo /etc/micronets/mso-portal.d/mso-portal docker-run
   ```

0. Verify the MSO Portal started up successfully:

   ```
   /etc/micronets/mso-portal.d/mso-portal docker-logs
   ```

   - You should see output like the following at the end of the log:

    ```
    Feathers application started on "http://0.0.0.0:3210"
    Feathers  webSocketBaseUrl "wss://my-server.org:5050/micronets/v1/ws-proxy/gw"
    Feathers  publicApiBaseUrl "https://my-server.org:443"
    ```

0. To securely expose the MSO API, configure your https proxy to redirect to localhost port
   3210. For example, an nginx configuration in `/etc/nginx/sites-enabled/micronets-mso-portal` 
   managed by Let's Encrypt could contain:
   
   ```
   # Micronets MSO Portal API
   server {
        server_name example.com;
    
        location /micronets/mso-portal/ {
            proxy_pass http://127.0.0.1:3210/;
        }
   }
   ```
