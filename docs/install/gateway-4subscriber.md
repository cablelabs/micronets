## Adding a Micronets MSO Subscriber:

### for NCCoE Build 3

0. Create a gateway configuration file for the Micronets gateway to register
   for the subscriber

    The wireless and wired interfaces configured in the gateway's `/etc/network/interfaces`
    file, the MAC addresses of the configured interfaces, and the available subnets are all 
    provided in the gateway configuration file. 
    
    For example, if the gateway has one wireless interface ("wlp2s0" with MAC address 
    "c0:d9:62:ae:f0:c2") and one wired interface ("wlp2s0" with MAC address "c0:d9:62:ae:f0:c2"),
    a gateway configuration like the following could be registered for a subscriber:
    
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

0. Registering a gateway configuration for a subscriber:

    If the gateway configuration is saved into a file called `gateway-config-001.json`,
    it can be registered with "subscriber-001" using:
    
    ```
    curl -s -X POST https://my-server.org/sub/subscriber-001/api/mm/v1/micronets/odl \
     -H "Content-Type: application/json" -d @./gateway-config-001.json | json_pp
    ```
    
    The applied gateway config should be returned if successfully applied.
    
0. Confirm that the Micronets Manager has registered with the MSO Portal:

   After the gateway configuration is applied, the Micronets Manager should have
   registered itself with the MSO Portal. To confirm, the following endpoint can 
   be invoked:
   
   ``` 
   curl -s https://my-server.org/micronets/mso-portal/portal/v1/subscriber/subscriber-001 | json_pp
   ```
   
   which should return a json object with a populated `registry` field that refers back
   to the Micronets Manager instance for the subscriber. e.g.:

   ```
    {
       "gatewayId" : "micronets-gw",
       "registry" : "https://my-server.org/sub/subscriber-001/api",
       "id" : "subscriber-001",
       "name" : "Subscriber 001",
       "ssid" : "micronets-gw"
    }
   ```
