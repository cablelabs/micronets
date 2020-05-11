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
   curl -s -L -O https://github.com/docker/compose/releases/download/1.24.1/docker-compose-Linux-`uname -m`
   sudo install -v -o root -m 755 docker-compose-Linux-`uname -m` /usr/local/bin/docker-compose
   ```

0. Download the management script:

   ```
   curl -s -O https://raw.githubusercontent.com/cablelabs/micronets-manager/nccoe-build-3/scripts/mm-container
   curl -s -O https://raw.githubusercontent.com/cablelabs/micronets-manager/nccoe-build-3/scripts/docker-compose.yml
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

0. Copy the shared secret value generated during the MSO Portal install:

   ```
   sudo install -v -o root -g docker -m 660 -D -t /etc/micronets/micronets-manager.d/lib mso-auth-secret
   ```

    Note: This is just one example of how to copy the `mso-auth-secret` file from server to server.

0. Download the Micronets Manager docker image:

   ```
   /etc/micronets/micronets-manager.d/mm-container pull
   ```

    Note: If you cannot connect to the Docker service, use "sudo usermod -aG docker <username>" to
          add the user account to the docker group.

0. Configure nginx for Micronets Manager:

   The Micronets Manager management script creates nginx forward entries that 
   provide a unique URI for each MM docker image. To create the infrastructure for
   these entries, run:

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
        server_name my-server.org;
    
        include /etc/nginx/micronets-subscriber-forwards/*.conf;
   }
   ```

   Note that it's not necessary to restart nginx after these changes 
   since the `mm-container` script will cause nginx to reload files.
   But if you want to ensure your edits are OK, you can run 
   `sudo nginx -s reload` to have nginx reload the configuration files.

0. Configure Micronets Manager to communicate with other Micronets services:

    A number of MM settings need to be adjusted in the 
    `/etc/micronets/micronets-manager.d/docker-compose.yml` file to enable interoperation 
    with the other Micronets services. The settings can be found in the `services/api/environment` 
    block of the `docker-compose.yml` file. For instance, if all the Micronets
    services are running on the same server, edit the `docker-compose.yml` and set the 
    variables to the following:
    
    ```
      MM_API_PUBLIC_BASE_URL: https://my-server.org/sub/${MM_SUBSCRIBER_ID}/api
      MM_APP_PUBLIC_BASE_URL: https://my-server.org/sub/${MM_SUBSCRIBER_ID}/app
      MM_MSO_PORTAL_BASE_URL: https://my-server.org/micronets/mso-portal
      MM_MUD_MANAGER_BASE_URL: https://my-server.org/micronets/mud-manager
      MM_MUD_REGISTRY_BASE_URL: https://my-server.org/micronets/mud/v1
      MM_GATEWAY_WEBSOCKET_BASE_URL: wss://my-server.org:5050/micronets/v1/ws-proxy/gw
    ```

    * Note: Micronets Managers are created per-subscriber. So validation of 
            these settings can be performed [after adding the first subscriber in the 
            Operations Guide](../operation/subscriber-setup.md).