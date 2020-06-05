## Installation guide for the Micronets MUD Manager

### for NCCoE Build 3

Prerequisites:

1. A Ubuntu 18.04 LTS server reachable by the server hosting the Micronet Manager instances
and any Micronets gateways
2. Docker (v18.06 or higher)
4. curl

Instructions:

0. Download the management script:

   ```
   curl -O https://raw.githubusercontent.com/cablelabs/micronets-mud-tools/nccoe-build-3/bin/micronets-mud-manager
   sudo install -v -o root -m 755 -D -t /etc/micronets/micronets-mud-manager.d/ micronets-mud-manager
   ```

    Note: These instructions assume the default values contained in the management script.
    These values can be modified directly in your copy of the management script or overridden via command-line
    parameters.

0. Configure the management script for your setup:

   Confirm and/or modify the default variables in the management script.
   For example:

   ```
    DEF_MUD_CACHE_PATH=/var/cache/micronets-mud
    DEF_BIND_PORT=8888
    DEF_BIND_ADDRESS=127.0.0.1
    DEF_CONTROLLER_ADDRESS=my-server.org
   ```

0. Download the docker image:

   ```
   /etc/micronets/micronets-mud-manager.d/micronets-mud-manager docker-pull
   ```

    Note: If you cannot connect to the Docker service, use "sudo usermod -aG docker <username>" to
          add the user account to the docker group.

0. Setup the MUD cache directory:

   ```
   /etc/micronets/micronets-mud-manager.d/micronets-mud-manager setup-cache-dir
   ```

0. Start the MUD manager:

   ```
   /etc/micronets/micronets-mud-manager.d/micronets-mud-manager docker-run
   ```

0. Verify the MUD Manager is running:

   ```
   /etc/micronets/micronets-mud-manager.d/micronets-mud-manager docker-logs
   ```

   - You should see output like the following:
   
    ```
    micronets-mud-manager: INFO Bind address: 0.0.0.0
    micronets-mud-manager: INFO Bind port: 8888
    micronets-mud-manager: INFO CA path: /etc/ssl/certs
    micronets-mud-manager: INFO Additional CA certs: None
    micronets-mud-manager: INFO MUD cache directory: /mud-cache-dir
    micronets-mud-manager: INFO Controller: None
    Running on https://127.0.0.1:8888 (CTRL + C to quit)
    ```

0. Setup the nginx entry to forward HTTPS requests to the MUD Manager by adding the following to the nginx `server` block:

```
    location /micronets/mud-manager/ {
        proxy_pass      http://localhost:8888/;
    }
```

Mmke sure to have nginx reload config files after modification:

```
sudo nginx -s reload
```

0. Test the MUD Manager by retrieving a MUD URL:

   ```
   curl -q -X POST -H "Content-Type: application/json" \
    https://my.server.org/micronets/mud-manager/getMudFile \
    -d '{"url": "https://alpineseniorcare.com/micronets-mud/ciscopi.json"}' 
   ```

   - You should see output like the following:
   
   ```
    {
        "ietf-mud:mud": {
            "mud-version": 1,
            "mud-url": "https://mudfileserver/ciscopi2",
            "last-update": "2018-12-05T19:42:01+00:00",
            "cache-validity": 24,
            "is-supported": true,
            "systeminfo": "ingress/egress ",
            . . .
   ```

   And the MUD cache directory should contain a copy of the retrieved MUD file. e.g.
   
   ```
    $ ls -1  /var/cache/micronets-mud/
    alpineseniorcare.com_micronets-mud_ciscopi.json
    alpineseniorcare.com_micronets-mud_ciscopi.json.md
   ```

   The cache cah be cleared by running:
   
   ```
   /etc/micronets/micronets-mud-manager.d/micronets-mud-manager clear-cache-dir
   ```
