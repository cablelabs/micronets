## Installation guide for the Micronets Websocket Proxy

### for NCCoE Build 3

Prerequisites:

1. A server reachable by the server hosting the Micronet Manager instances
and any Micronets gateways
2. Docker and Docker-compose
3. OpenSSL
4. curl

Instructions:

0. Download the cert generation scripts:

   ```
   curl -O https://github.com/cablelabs/micronets-ws-proxy/blob/nccoe-build-3/bin/gen-root-cert
   curl -O https://github.com/cablelabs/micronets-ws-proxy/blob/nccoe-build-3/bin/gen-leaf-cert
   ```

1. Create the root certificate for the WS proxy:

   ```
   ./gen-root-cert --cert-basename micronets-ws-root \
       --subject-org-name "Micronets Websocket Root Cert" \
       --expiration-in-days 3650
   ```

2. Create the WS proxy's server certificate & private key:

   - This certificate and key is used to host the websocket proxy server.

   ```
   ./gen-leaf-cert --cert-basename micronets-ws-proxy \
      --subject-org-name "Micronets Websocket Proxy Cert" \
      --expiration-in-days 3650 \
      --ca-certfile micronets-ws-root.cert.pem \
      --ca-keyfile micronets-ws-root.key.pem
   cat micronets-ws-proxy.cert.pem micronets-ws-proxy.key.pem \
      > micronets-ws-proxy.pkeycert.pem   ```
   ```

3. Generate the certificate & key to be used by the Micronets Manager to connect to the WS Proxy:

   - These files will be used to enable the Micronets Manager to connect to the proxy.

   ```
    ./gen-leaf-cert --cert-basename micronets-manager \
        --subject-org-name "Micronets Manager Websocket Client Cert" \
        --expiration-in-days 3650 \
        --ca-certfile micronets-ws-root.cert.pem \
        --ca-keyfile micronets-ws-root.key.pem

    cat micronets-manager.cert.pem micronets-manager.key.pem \
        > micronets-manager.pkeycert.pem
   ```

4. Generate the certificate to be used by the Micronets Gateway to connect to the WS Proxy:

   - These files will be used to enable the Micronets Gateway to connect to the proxy.

   ```
    ./gen-leaf-cert --cert-basename micronets-gateway-service \
        --subject-org-name "Micronets Gateway Service Websocket Client Cert" \
        --expiration-in-days 3650 \
        --ca-certfile micronets-ws-root.cert.pem \
        --ca-keyfile micronets-ws-root.key.pem

    cat micronets-gateway-service.cert.pem micronets-gateway-service.key.pem \
        > micronets-gateway-service.pkeycert.pem
   ```

5. Download the management script:

   ```
   sudo mkdir -p /etc/micronets.d/micronets-ws-proxy/lib
   cd /etc/micronets.d/micronets-ws-proxy
   curl -O https://github.com/cablelabs/micronets-ws-proxy/blob/nccoe-build-3/bin/micronets-ws-proxy.sh
   ```

6. Copy the ws proxy server cert and key for use by the WS Proxy Docker container:

   ```
   sudo cp micronets-ws-proxy.pkeycert.pem /etc/micronets.d/micronets-ws-proxy/lib/
   sudo cp micronets-ws-root.cert.pem /etc/micronets.d/micronets-ws-proxy/lib/
   ```

7. Download the Micronets Websocket proxy image:

   ```
   /etc/micronets.d/micronets-ws-proxy/micronets-ws-proxy.sh docker-pull
   ```

8. Start the proxy:

   ```
   /etc/micronets.d/micronets-ws-proxy/micronets-ws-proxy.sh docker-run
   ```

9. Verify the Websocket Proxy is running:

   ```
   /etc/micronets.d/micronets-ws-proxy/micronets-ws-proxy.sh docker-logs
   ```

   - You should see output like the following:
   
    ```
    micronets-ws-proxy: INFO Bind address: 0.0.0.0
    micronets-ws-proxy: INFO Bind port: 5050
    micronets-ws-proxy: INFO Server cert/key: lib/micronets-ws-proxy.pkeycert.pem
    micronets-ws-proxy: INFO CA path: None
    micronets-ws-proxy: INFO Additional CA certs: lib/micronets-ws-root.cert.pem
    micronets-ws-proxy: INFO URL Path Prefix: /micronets/v1/ws-proxy/
    micronets-ws-proxy: INFO Report Interval: 0
    micronets-ws-proxy: INFO Loading proxy certificate/key from lib/micronets-ws-proxy.pkeycert.pem
    micronets-ws-proxy: INFO Starting micronets websocket proxy on 0.0.0.0 port 5050...
    micronets-ws-proxy: INFO Clients may connect to wss://0.0.0.0:5050/micronets/v1/ws-proxy/*
    ```

10. Save the `micronets-manager.pkeycert.pem`, `micronets-gateway-service.pkeycert.pem`,
   and `micronets-ws-root.cert.pem` files for configuring the Micronets Manager
   and Micronet Gateway components.
