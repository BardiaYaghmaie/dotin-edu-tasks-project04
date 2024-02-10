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
sudo docker run -d --privileged --name dind docker:dind
sudo docker run -d --privileged --name dind-2 docker:dind
sudo docker run -d --privileged --name dind-3 docker:dind

```

2- Enter the "dind" container, Inside the "dind" container:
```
docker swarm init --advertise-addr <IP_of_manager_node>
```

3- Login to "dind-2" and "dind-3" containers. Inside them:
```
docker swarm join --token <token> <IP_of_manager_node>:2377

```



