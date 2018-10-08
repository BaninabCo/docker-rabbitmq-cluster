Docker images to run RabbitMQ cluster with dynamic number of nodes.

# Building

Once you clone the project run this command to build the custom image:

```
sh build.sh
```

# Running with docker-compose

If you want to run the cluster on one machine use [docker-compose](https://github.com/docker/compose/)

```
docker-compose up -d
```

By default 4 nodes are started up this way:

```
version: '2.3'

services:
    lb:
        image: dockercloud/haproxy
        ports:
          - '5672:5672'
        networks:
          - dcmon-net
        volumes:
          - /var/run/docker.sock:/var/run/docker.sock
        environment:
          - TIMEOUT=connect 5000, client 0, server 0
        links:
          - rabbitmq

    rabbitmq-master:
        hostname: rabbitmq-master
        image: rabbitmq-cluster:3.7.5-rc.1
        environment:
          - RABBITMQ_DEFAULT_USER=admin
          - RABBITMQ_DEFAULT_PASS=admin
          - ERLANG_COOKIE=SOMETHING_BLAH_BLAH_SECURE
        ports:
          - "15672:15672"
        expose:
            - "5672"
            - "25672"
            - "4369"
            - "9100"
            - "9101"
            - "9102"
            - "9103"
            - "9104"
            - "9105"
    rabbitmq:
        scale: 4
        environment:
          - CLUSTER_WITH=rabbitmq-master
          - TCP_PORTS=5672
          - ERLANG_COOKIE=SOMETHING_BLAH_BLAH_SECURE
        expose:
            - "5672"
            - "25672"
            - "4369"
            - "9100"
            - "9101"
            - "9102"
            - "9103"
            - "9104"
            - "9105"
```

If you need more or less node, just change the scale number:

```
    rabbitmq:
        scale: 5
```

Once cluster is up:
* The management console can be accessed at `http://hostip:15672`
* The HA proxy manages the load balancing automatically. it manages the connections throgh 5672.
