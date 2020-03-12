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
`    curl 
```
2. Modify the Makefile
3. Download the Docker image

```
    make docker-pull
```

4. Start the Docker container

```
    make docker-run
```
