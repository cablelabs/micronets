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
   curl -O https://raw.githubusercontent.com/cablelabs/micronets-ws-proxy/master/bin/gen-root-cert
   curl -O https://raw.githubusercontent.com/cablelabs/micronets-ws-proxy/master/bin/gen-leaf-cert
   chmod +x gen-root-cert gen-leaf-cert
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
      > micronets-ws-proxy.pkeycert.pem
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
   curl -O https://raw.githubusercontent.com/cablelabs/micronets-ws-proxy/master/bin/micronets-ws-proxy
   sudo install -v -o root -m 755 -D -t /etc/micronets/ micronets-ws-proxy 
   ```

6. Copy the ws proxy server cert and key for use by the WS Proxy Docker container:

   ```
   sudo install -v -o root -m 600 -D -t /etc/micronets/micronets-ws-proxy.d/lib micronets-ws-proxy.pkeycert.pem  micronets-ws-root.cert.pem 
   ```

7. Download the Micronets Websocket proxy image:

   ```
   /etc/micronets/micronets-ws-proxy docker-pull
   ```

8. Start the proxy:

   ```
   /etc/micronets.d/micronets-ws-proxy docker-run
   ```

9. Verify the Websocket Proxy is running:

   ```
   /etc/micronets/micronets-ws-proxy docker-logs
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

10. Verify the websocket proxy credentials

   ```
   curl -O https://raw.githubusercontent.com/cablelabs/micronets-ws-proxy/nccoe-build-3/bin/websocket-test-client.py
   curl -O https://raw.githubusercontent.com/cablelabs/micronets-ws-proxy/nccoe-build-3/requirements.txt
   virtualenv --clear -p $(which python3.6) $PWD/virtuanenv
   ./virtuanenv/bin/pip install -r requirements.txt
   ./virtuanenv/bin/python websocket-test-client.py \
        --client-cert micronets-manager.pkeycert.pem \
        --ca-cert micronets-ws-root.cert.pem  \
        wss://localhost:5050/micronets/v1/ws-proxy/test/mm
   
   ```

   if everything is configured correctly, the test client should output:
   
   ```
    Startup...
    Loading test client certificate from micronets-manager.pkeycert.pem
    Loading CA certificate from micronets-ws-root.cert.pem
    ws-test-client: Opening websocket to wss://localhost:5050/micronets/v1/ws-proxy/test/mm...
    ws-test-client: Connected to wss://localhost:5050/micronets/v1/ws-proxy/test/mm.
    ws-test-client: Sending HELLO message...
    ws-test-client: > sending hello message:  {"message": {"messageId": 0, "messageType": "CONN:HELLO", "requiresResponse": false, "peerClass": "micronets-ws-test-client", "peerId": "12345678"}}
    ws-test-client: Waiting for HELLO message...
   ```

   and `/etc/micronets/micronets-ws-proxy docker-logs` should include:

   ```
    2020-03-25T09:48:01.742413418Z 2020-03-25 09:48:01,742 micronets-ws-proxy: INFO ws_connected: client 140030650222128: Received HELLO message:
    2020-03-25T09:48:01.742422736Z 2020-03-25 09:48:01,742 micronets-ws-proxy: INFO {
    2020-03-25T09:48:01.742426214Z   "message": {
    2020-03-25T09:48:01.742429177Z     "messageId": 0,
    2020-03-25T09:48:01.742432027Z     "messageType": "CONN:HELLO",
    2020-03-25T09:48:01.742434882Z     "peerClass": "micronets-ws-test-client",
    2020-03-25T09:48:01.742437762Z     "peerId": "12345678",
    2020-03-25T09:48:01.742440568Z     "requiresResponse": false
    2020-03-25T09:48:01.742443356Z   }
    2020-03-25T09:48:01.742445991Z }
   ```

11. Save the `micronets-manager.pkeycert.pem`, `micronets-gateway-service.pkeycert.pem`,
   and `micronets-ws-root.cert.pem` files for configuring the Micronets Manager
   and Micronet Gateway components.
