## Installation guide for the Micronets Manager

### for NCCoE Build 3

Prerequisites:

1. A Ubuntu 18.04 LTS server reachable by the server hosting the Micronet Manager instances
and any Micronets gateways
2. Docker and Docker-compose
3. OpenSSL
4. curl

Instructions:

1. Download the management script:

   ```
   curl -O https://raw.githubusercontent.com/cablelabs/micronets-manager/nccoe-build-3/scripts/mm-container
   curl -O https://raw.githubusercontent.com/cablelabs/micronets-manager/nccoe-build-3/scripts/docker-compose.yml
   sudo install -v -o root -m 755 -D -t /etc/micronets/micronets-manager.d mm-container
   sudo install -v -o root -m 644 -D -t /etc/micronets/micronets-manager.d docker-compose.yml
   ```

    Note: These instructions assume the default values contained in the mm-container management script.
    These values can be modified directly in your copy of the management script or overridden via command-line
    parameters.

2. Copy the mm server cert/key and the WS Proxy root CA cert created in earlier steps
   for use by the Micronets Manager docker container(s):

   ```
   sudo install -v -o root -m 600 -D -t /etc/micronets/micronets-manager.d/lib micronets-manager.{cert,key}.pem micronets-ws-root.cert.pem
   sudo touch /etc/micronets/micronets-manager.d/lib/micronets-ws-proxy.pkeycert.pem
   ```

   Note: Creating the `micronets-ws-proxy.pkeycert.pem` file is a work-around for an issue
   that will be fixed in the near future.

3. Generate a shared secret for communicating between the MSO Portal and the MM:

   ```
   sudo /etc/micronets/micronets-manager.d/mm-container create-mso-secret
   ```

    Note: This may be changed to generate the shared secret during the MSO Portal
          installation.

4. Download the Micronets Manager docker image:

   ```
   /etc/micronets/micronets-manager.d/mm-container pull
   ```

    Note: If you cannot connect to the Docker service, use "sudo usermod -aG docker <username>" to
          add the user account to the docker group.

5. Start a Micronets Manager for a subscriber:

   ```
   /etc/micronets/micronets-manager.d/mm-container start <subscriber-name>
   ```

6. Verify the Micronets Manager is running:

   ```
   /etc/micronets/micronets-manager.d/mm-container logs <subscriber>
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
