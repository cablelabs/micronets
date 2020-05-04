## Removing a Micronets MSO Subscriber:

### for NCCoE Build 3

Removing a subscriber is a pretty simple process.

0. Remove the subscriber from the MSO Portal using:

    ```
    curl -s -X DELETE https://my-server.org/micronets/mso-portal/portal/v1/subscriber/subscriber-001
    ```

0. Remove the Micronets Manager instance for a subscriber by running:

    ```
    /etc/micronets/micronets-manager.d/mm-container delete subscriber-001
    ```

    This will terminate the server, destroy the associated database, and
    remove any nginx forwarding rules. e.g.
    
    ```
    Deleting resources for subscriber subscriber-001...
    Stopping sub-subscriber-001_api_1     ... done
    Stopping sub-subscriber-001_mongodb_1 ... done
    Removing sub-subscriber-001_api_1     ... done
    Removing sub-subscriber-001_mongodb_1 ... done
    Removing network sub-subscriber-001_mm-priv-network
    Removing volume sub-subscriber-001_mongodb
    removed '/etc/nginx/micronets-subscriber-forwards/sub-subscriber-001.conf'
    Issuing nginx reload (running 'sudo nginx -s reload')
    ```



Add a subscriber and associated user account and password to the MSO Portal:

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
       "ssid" : "micronets-gw",
       "id" : "subscriber-001",
       "gatewayId" : "micronets-gw",
       "name" : "Subscriber 001",
       "registry" : ""
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

   Note that no Micronet Managers will be running until a subscriber is added to the 
   MSO Portal database. Once one is added, the health of a MM instance can be checked 
   using:

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
