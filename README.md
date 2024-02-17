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
3- Change docker-compose.yml od project03 to be able to work with docker swarm

new docker-compose.yml
```
version: '3'
services:

  accounts:
    image: pr03-account:1
    ports:
      - "8000:80"
    container_name: pr03-account
    hostname: account
    networks:
      - project03-overlay


  shop:
    image: pr03-shop:1
    ports:
      - "8001:80"
    container_name: pr03-shop
    hostname: shop
    networks:
      - project03-overlay


  order:
    image: pr03-order:1
    ports:
      - "8002:80"
    container_name: pr03-order
    hostname: order
    networks:
      - project03-overlay


  nginx:
    image: pr03-nginx:3
    ports:
      - "9999:80"
    container_name: pr03-nginx
    networks:
      - project03-overlay
    depends_on:
      - accounts
      - shop
      - order


  haproxy:
    image: pr03-haproxy:1
    ports:
      - "7777:80"
      - "8404:8404"
    container_name: pr03-haproxy
    networks:
      - project03-overlay

networks:
  project03-overlay:
    driver: overlay
```



