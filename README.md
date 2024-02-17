# Problem

## Step 1
Deploy a 3-node Docker Swarm cluster using Docker in Docker (DinD)

## Step 2
Deploy your previous project Stack on this Docker Swarm using the swarm stack file.

## Step 3
Enable rolling updates for all services.

## Step 4
Set up Prometheus and Grafana monitoring stack for your swarm cluster. You should collect your containers’ metrics and visualize them using Grafana dashboards.

------
# Solution

## Step 1

0- First we clone project03 and change names to 04, then we change static ipv4s to container names / service names 

1- Start 3 Docker containers with Docker in Docker enabled
```
sudo docker run -d --privileged --name dind1 docker:dind
sudo docker run -d --privileged --name dind2 docker:dind
sudo docker run -d --privileged --name dind3 docker:dind
```

2- Login to dind containers. Inside them:
```
docker swarm join --token <token> <IP_of_manager_node>:2377

```
## Step 2

3- Change docker-compose.yml and other configs of project03 to be able to work with docker swarm

new docker-compose.yml
```
version: '3'
services:

  accounts:
    image: pr04-account:1
    ports:
      - "8000:80"
    container_name: pr04-account
    hostname: account
    networks:
      - project04-overlay


  shop:
    image: pr04-shop:1
    ports:
      - "8001:80"
    container_name: pr04-shop
    hostname: shop
    networks:
      - project04-overlay


  order:
    image: pr04-order:1
    ports:
      - "8002:80"
    container_name: pr04-order
    hostname: order
    networks:
      - project04-overlay


  nginx:
    image: pr04-nginx:2
    ports:
      - "9999:80"
    container_name: pr04-nginx
    networks:
      - project04-overlay
    depends_on:
      - accounts
      - shop
      - order


  haproxy:
    image: pr04-haproxy:1
    ports:
      - "7777:80"
      - "8404:8404"
    container_name: pr04-haproxy
    networks:
      - project04-overlay
    depends_on:
      - nginx

networks:
  project04-overlay:
    driver: overlay
```

new pr04.conf
```
server {
.
.
.
    location /order {
        proxy_pass http://project04_order:80;
    }
}
```

new haproxy.cfg
```
.
.
.
backend nginx-backend
    balance roundrobin
    server nginx1 project04_nginx:80 check
    server nginx2 project04_nginx:80 check
    server nginx3 project04_nginx:80 check
```

4- Deploy the stack using:
```
 docker stack deploy -c docker-compose.yml project04
```

5- Access services via <IP-of-Manager-Node>:<Port>


