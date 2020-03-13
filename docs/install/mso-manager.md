## Installation guide for Micronets MSO Portal

### for NCCOE Build 3

Prerequisites:

1. A server reachable by the server hosting the Micronet Manager instances
and whatever device is running the mobile application.
2. Docker and Docker-compose
3. nginx and requisite certificates if https is to be supported

Instructions:

1. Download the MSO Portal docker Makefile

   ```
   curl -O  
   ```
2. Modify the Makefile:

   - DOCKER_REGISTRY should be set to `community.cablelabs.com:4567`
   - DOCKER_IMAGE_TAG should be `nccoe-build-3`

3. Download the Docker image

    ```
    make -f mso-portal.make docker-pull
    ```

4. Start the Docker container

    ```
    make -f mso-portal.make docker-run
    ```
