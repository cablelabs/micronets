## Adding a Micronets MSO Subscriber:

### for NCCoE Build 3

0. Add a subscriber and associated user account to the MSO Portal:

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

0. Confirm that the subscriber's Micronets Manager is registered with the MSO Portal:

    ```
    curl -s https://my0server.org/micronets/mso-portal/portal/v1/subscriber/subscriber-002 | json_pp
    ```
    
    which should return the :
    
    ```
    {
       "gatewayId" : "micronets-gw",
       "registry" : "",
       "ssid" : "micronets-gw",
       "name" : "Subscriber 001",
       "id" : "subscriber-001"
    }
    ```

