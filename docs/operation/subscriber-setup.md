## Adding a Micronets MSO Subscriber:

### for NCCoE Build 3

0. Add a subscriber and associated user account and password to the MSO Portal:

    ```
    curl -s -X POST https://my-server.org/micronets/mso-portal/portal/v1/subscriber \
        -H "Content-Type: application/json" \
        -d '{
                "id" : "subscriber-001",
                "ssid" : "micronets-gw",
                "name" : "Subscriber 001",
                "gatewayId":"micronets-gw",
                "username":"micronets",
                "password":"micronets"
        }' \
    | json_pp
    ```

    if this executes successfully, it should return a 200 with the added subscriber's fields. e.g.:
    
    ```
    {
       "id" : "subscriber-001",
       "name" : "Subscriber 001",
       "registry" : "",
       "gatewayId" : "default-gw-subscriber-001",
       "ssid" : "micronets-gw"
    }
    ```

0. Start the Micronets Manager for the subscriber by running:

    ```
    /etc/micronets/micronets-manager.d/mm-container start subscriber-001
    ```
    
    this should produce output like the following:
    
    ```
    Creating resources for subscriber subscriber-001...
    Creating network "sub-subscriber-001_mm-priv-network" with the default driver
    Creating volume "sub-subscriber-001_mongodb" with default driver
    Creating sub-subscriber-001_mongodb_1 ... done
    Creating sub-subscriber-001_api_1     ... done
    Issuing nginx reload (running 'sudo nginx -s reload')
    ```
    
    Note: The nginx reload requires `sudo`. So you may be prompted for a password.

0. You can confirm that the Micronets Manager started up successfully by running:

   ```
   /etc/micronets/micronets-manager.d/mm-container logs subscriber-001
   ```

   - You should see output like the following toward the top of the log:

    ```
    Feathers application started on "http://0.0.0.0:3030"
    Public base URL: "https://my-server.org/sub/subscriber-001/api"
    ...
    Mano web socket base url : "wss://my-server.org:5050/micronets/v1/ws-proxy/gw"
    MSO Portal url : "https://my-server.org/micronets/mso-portal"
    ```

0. Verify the Micronets Manager for the subscriber has registered with the MSO Portal:

   ``` 
   curl -s https://my-server.org/micronets/mso-portal/portal/v1/subscriber/subscriber-001 | json_pp
   ```

   which should return a json object with a populated `registry` field that refers back
   to the Micronets Manager instance for the subscriber. e.g.:

    ```
    {
       "id" : "subscriber-001",
       "name" : "Subscriber 001",
       "registry" : "https://my-server.org/sub/subscriber-001/api",
       "gatewayId" : "default-gw-subscriber-001",
       "ssid" : "micronets-gw"
    }
    ```