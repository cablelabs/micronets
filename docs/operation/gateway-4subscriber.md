## Adding a Micronets MSO Subscriber:

### for NCCoE Build 3

0. Create a gateway configuration file for the Micronets gateway to register
   for the subscriber

    The wireless and wired interfaces configured in the gateway's `/etc/network/interfaces`
    file, the MAC addresses of the configured interfaces, and the available subnets are all 
    provided in the gateway configuration file. 
    
    For example, if the gateway has one wireless interface ("wlp2s0" with MAC address 
    "c0:d9:62:ae:f0:c2") and one wired interface ("enx00e04c534458" with MAC address 
    "00:e0:4c:53:44:58") that you wanted to use for micronets, a gateway configuration 
    like the following could be registered for a subscriber:
    
    ```
    {
        "version": "1.0",
        "gatewayId": "micronets-gw",
        "gatewayModel": "proto-gateway",
        "gatewayVersion": {"major":1, "minor":0, "micro":0},
        "configRevision": 1,
        "vlanRanges": [
            {"min":1000, "max":4095}
        ],
        "micronetInterfaces": [
            {
                "medium": "wifi",
                "name": "wlp2s0",
                "macAddress": "c0:d9:62:ae:f0:c2",
                "ssid": "micronets-gw",
                "dpp": {
                    "supportedAkms": ["psk"]
                },
                "ipv4SubnetRanges": [
                    {
                        "id": "range001",
                        "subnetRange": {"octetA": 10,
                                        "octetB": 135,
                                        "octetC": {"min":1, "max":5}
                        },
                        "subnetGateway": {"octetD": 1},
                        "deviceRange": {"octetD": {"min":2, "max":254}}
                    }
                ]
            },
            {
                "medium": "ethernet",
                "name": "enx00e04c534458",
                "macAddress": "00:e0:4c:53:44:58",
                "ipv4Subnets": [
                    {
                        "id": "range001",
                        "subnetRange": {"octetA": 10,
                                        "octetB": 135,
                                        "octetC": 250
                        },
                        "subnetGateway": {"octetD": 1},
                        "deviceRange": {"octetD": {"min":2, "max":254}}
                    }
                ]
            }
        ]
    }
    ``` 
    
    Note that a wireless interface can support multiple micronets. In this case
    it's declared to support up to 5 "class C" micronets (10.135.1.x through 10.135.5.x).
    Wired interfaces can only support 1 micronet each. For all micronets defined by
    this gateway config, devices can be assigned addresses from 2-254 on each 
    micronet - with the gateway address on each set to 1.

0. Registering a gateway configuration for a subscriber:

    If the gateway configuration is saved into a file called `gateway-config-001.json`,
    it can be registered with "subscriber-001" using:
    
    ```
    curl -s -X POST https://my-server.org/sub/subscriber-001/api/mm/v1/micronets/odl \
    -H "Content-Type: application/json" -d @./gateway-config-001.json | json_pp
    ```

    This should return 200 and return applied gateway config when successfully applied.

    You can retrieve/check the gateway config using:
    
    ```
    curl -s https://my-server.org/sub/subscriber-001/api/mm/v1/micronets/odl/<gatewayId> \
    | json_pp
    ```

    Where `<gatewayId>` is the gateway ID specified in your gateway config file.

    You can delete the gateway config using:
 
    ```
    curl -s -X DELETE https://my-server.org/sub/subscriber-001/api/mm/v1/micronets/odl/<gatewayId> \
    | json_pp
    ```

    Where `<gatewayId>` is the gateway ID specified in your gateway config file.
    
    
0. Confirm that the gateway ID is updated in the MSO Portal:

   After the gateway configuration is applied, the Micronets Manager should have
   registered itself with the MSO Portal. To confirm, the following endpoint can 
   be invoked:
   
   ``` 
   curl -s https://my-server.org/micronets/mso-portal/portal/v1/subscriber/subscriber-001 | json_pp
   ```
   
   which should return a json object with a `gatewayId` field that matches the gatewayId
   in the Gateway Configuration file. e.g.:

    ```
    {
       "id" : "subscriber-001",
       "name" : "Subscriber 001",
       "registry" : "https://my-server.org/sub/subscriber-001/api",
       "gatewayId" : "micronets-gw",
       "ssid" : "micronets-gw"
    }
    ```

0. Configure the Micronets gateway with the WS Proxy keys provisioned for the gateway:

   First copy the client cert and key and the WS root certificate into the gateway. 
   Then copy them into the gateway service library to be loaded when the gateway is
   restarted.

   ```
   sudo cp -v micronets-gw-service.pkeycert.pem micronets-ws-root.cert.pem /opt/micronets-gw/lib/
   ```
   
   Note: This assumes the default name/location of the client cert and key files.
   The name/location of the gateway's client cert/key file and the root WS proxy 
   certificate file can be specified by modifying the `WEBSOCKET_TLS_CERTKEY_FILE` and
   `WEBSOCKET_TLS_CA_CERT_FILE` variables in the `/opt/micronets-gw/config.py` gateway 
   configuration file, if so desired.

0. Change the websocket lookup URL to use the MSO Portal service on your server:

   The Micronet gateway service config file `/opt/micronets-gw/config.py` contains
   the `WEBSOCKET_LOOKUP_URL` config variable, which needs to be set to the MSO Portal
   websocket lookup endpoint (see the [MSO Portal Install Instructions](../install/mso-portal.md)). 
   For example this can be set in the `BaseConfig` section of `config.py` using:
   
   ```
   WEBSOCKET_LOOKUP_URL = 'https://my-server.org/micronets/mso-portal/portal/v1/socket?gatewayId={gateway_id}'
   ```

0. Set the Gateway ID:

   The Micronet gateway's ID will default to the short machine name (run `hostname -s`
   to see the machine name). If the machine name does not
   match the gateway ID provided above ("micronets-gw") you can set the Gateway ID via the
   `GATEWAY_ID` in the Micronet gateway service config file `/opt/micronets-gw/config.py`.
   For example this can be set in the `BaseConfig` section of `config.py` using:

   ```
   GATEWAY_ID = 'micronets-gw'
   ```

0. Restart the Micronets gateway service:

   ```
   sudo systemctl restart micronets-gw.service 
   ```

0. Check the Micronets Gateway Service log to check the gateway's websocket registration status:

   In the file `/opt/micronets-gw/micronets-gw.log`, the following entries indicate a successful
   resolution of the websocket URL and successful authentication of the gateway's websocket
   certificate & key:

   ```
    WSConnector: initializing...
    WSConnector: Websocket URL (None=URL Lookup): None
    WSConnector: Client cert-key File: /opt/micronets-gw/lib/micronets-gw-service.pkeycert.pem
    WSConnector: CA File: /opt/micronets-gw/lib/micronets-ws-root.cert.pem
    WSConnector: Registering handler for message type prefix DPP: <app.dpp_handler.DPPHandler object at 0x7f06b61b2710>
    WSConnector: setup_connection: starting...
    WSConnector: get_websocket_url_for_gateway(grandpa-gw)...
    WSConnector: get_websocket_url_for_gateway(grandpa-gw): Retrieving https://my-server.org//micronets/mso-portal/portal/v1/socket?gatewayId=micronets-gw
    WSConnector: get_websocket_url_for_gateway(grandpa-gw): Received response: {'socketUrl': 'wss://my-server.org:5050/micronets/v1/ws-proxy/gw/subscriber-001', 'subscriberId': 'subscriber-001', 'gatewayId': 'micronets-gw'}
    WSConnector: get_websocket_url_for_gateway(grandpa-gw): Received URL: wss://my-server.org:5050/micronets/v1/ws-proxy/gw/subscriber-001
    WSConnector: init_connect opening wss://my-server.org:5050/micronets/v1/ws-proxy/gw/subscriber-001...
    WSConnector: init_connect opened wss://my-server.org:5050/micronets/v1/ws-proxy/gw/subscriber-001.
    WSConnector Sending HELLO message...
    WSConnector: Waiting for HELLO messages...
    WSConnector: HELLO handshake complete.

   ```
   
0. Establishment of the gateway-manager control connection can also be confirmed by examining
   the websocket proxy connection reports in the WS Proxy log.
   
   To look at WS Proxy logs, run the following on the websocket proxy server:
   
   ```
   /etc/micronets/micronets-ws-proxy docker-logs | less   
   ```
   
   Look for the following in the log (with the "MEETUP ID" matching the subscriber name in question):
   
   ```
    WEBSOCKET MEETUP TABLE REPORT FOR 0.0.0.0:5050//micronets/v1/ws-proxy/
    
     MEETUP ID: gw/subscriber-001
       Client 1: Client 140604181239960 (peer: gw service 139666802147224) @ ('50.39.97.46', 59672))
       Client 2: Client 140604181241584 (peer: 12345678) @ ('172.17.0.1', 36270))
   ```

   This indicates that the micronets gateway service and the micronets manager for the subscriber 
   and connected and can exchange provisioning commands and event indications.
   