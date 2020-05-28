## Removing a Micronets MSO Subscriber:

### for NCCoE Build 3

Removing a subscriber involves removing the subscriber from the MSO Portal database, 
removing the subscriber's micronets, and removing the subscriber's Micronets Manager.

0. Remove the subscriber from the MSO Portal using:

    ```
    curl -s -X DELETE https://my-server.org/micronets/mso-portal/portal/v1/subscriber/subscriber-001
    ```

0. Verify the subscriber is removed from the MSO Portal:

    ```
    curl -s https://my-server.org/micronets/mso-portal/portal/v1/subscriber/subscriber-001
    curl -s https://my-server.org/micronets/mso-portal/portal/v1/users/subscriber-001
    curl -s https://my-server.org/micronets/mso-portal/portal/v1/socket/subscriber-001
    ```

    Note: If there are issues deleting the subscriber from the subtables (if the above queries
    return non-empty results - e.g. due to issues communicating with the peer Micronets Manager), 
    the subscriber entries in the mso-portal subtables can be deleted using:

    ```
    curl -s -X DELETE https://my-server.org/micronets/mso-portal/portal/v1/users/subscriber-001
    curl -s -X DELETE https://my-server.org/micronets/mso-portal/portal/v1/socket/subscriber-001
    ```

0. Remove all the Micronets for the subscriber using:

    ```
    curl -s -X DELETE https://my-server.org/sub/subscriber-001/api/mm/v1/subscriber/subscriber-001/micronets
    ```

0. You can verify the Micronets have been deleted by running:

    ```
    curl -s https://my-server.org/sub/subscriber-001/api/mm/v1/subscriber/subscriber-001/micronets
    ```
    
    Which should return an empty `micronets` list. e.g.
    
    ```
    {
       "skip" : 0,
       "data" : [
          {
             "micronets" : [],
             "ssid" : "micronets-gw",
             "name" : "Subscriber 001",
             "gatewayId" : "micronets-gw",
             "id" : "subscriber-001",
             ...
          }
       ],
       "total" : 1,
       "limit" : 500
    }
    ```

0. Remove the Micronets Manager docker container for a subscriber by running:

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

0. You can confirm the subscriber MM is removed by running:

    ```
    curl -s https://my-server.org/sub/subscriber-001/api/mm/v1/subscriber/subscriber-001
    ```

    This should return a 404 error if the MM for the subscriber is successfully removed.
