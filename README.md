# Set-up-IP-geolocation-using-docker-swarm

Geolocation refers to the use of location technologies such as GPS or IP addresses to identify and track the whereabouts of connected electronic devices. Because these devices are often carried on an individual's person, geolocation is often used to track the movements and location of people and surveillance.

It is same as my previous git article regarding [multitier-ipgeolocation-caching-docker-compose](https://github.com/radin-lawrence/multitier-ipgeolocation-caching-docker-compose), But here we use docker swarm and in docker swarm we use service instead of container. Also, here we use 1 master with 2 worker instances.

## Pre-requests
- Running 3 Amazon Linux instance with docker installed
- A domain name to point the EC2 instance
- An [API_KEY](https://app.ipgeolocation.io/auth/login)


##  Initialising docker swarm

 Make sure the Docker Engine daemon is started on the host machines.

Open a terminal and ssh into the machine where you want to run your manager node. We need to initialize the docker swarm in the manager server, then we will get the generated join-token to add the workers to this swarm.

```bash
 docker swarm init
```

![image](https://user-images.githubusercontent.com/100775027/166470140-7d0ee8cf-e5aa-4e72-9e43-656ee9f97f90.png)


If you don’t have the command available, you can run the following command on a manager node to retrieve the join command for a worker:

```bash
docker swarm join-token worker
```

Run this generated join token in both of your worker instnaces:

```bash
docker swarm join --token  <YOURS TOKEN>
```

To view information about nodes and verify the workers has been added to master:

```bash
dokcer node ls
```
![image](https://user-images.githubusercontent.com/100775027/166471465-fbb8064e-9628-4cb3-bf6e-82e9cb7ffeab.png)

## Drain manager node on the swarm

The swarm manager can assign tasks to any ACTIVE node, so up to now, all nodes have been available to receive tasks. When we create a service in master, the master itself acts as a worker, therefore to avoid it we run drain master server. So to drain the master server, we need to change the availability state to drain:
```bash
docker node update --availability drain {master-server-hostname}
docker node ls
 ```
![image](https://user-images.githubusercontent.com/100775027/166471158-3a13a7e0-8641-48ff-8934-047edb380d13.png)


## Label the worker instances

We use Labels to add extra metadata to your docker images, service, networks, swarm nodes etc. Once added these label(s) allows you to filter your docker resources.
~~~bash
docker node update --label-add resource=cache {worker-server-hostname}
docker node update --label-add resource=disk {worker-server-hostname}
~~~
![image](https://user-images.githubusercontent.com/100775027/166471032-40875a8f-3b6c-45a4-9029-1244f04f3fba.png)

> Here I use 2 worker instnaces so am adding lables for those two. You can use 3 worker instances, like 1 for redis image, 1 for api-service image, and 1 for frontend image.

## Create an overlay network

Overlay networks connect multiple Docker daemons together and enable swarm services to communicate with each other.
```bash
docker network create --driver overlay <network name>
```
> An overlay network called ingress, which handles the control and data traffic related to swarm services. When you create a swarm service and do not connect it to a user-defined overlay network, it connects to the ingress network by default.

## Create services

**Redis cache**

It's for caching and here I'm usin the redis latest version image.

```bash
docker service create \
--name iplocation-cache-service \
--replicas 1 \
--network iplocation-net \
--constraint 'node.labels.resource == cache' \
redis:latest
```
![image](https://user-images.githubusercontent.com/100775027/166470934-45c0b730-cd99-488a-842a-f82636cce16b.png)

The --replicas flag specifies the desired state of 1 running instance.
Here my network name is iplocation-net,please replace it with your network name.

**API Layer**

```bash
docker service create \
--name iplocation-api-service \
--network iplocation-net \
--replicas 4 \
--constraint 'node.labels.resource == disk' \
-e REDIS_PORT="6379" \
-e REDIS_HOST="iplocation-cache-service" \
-e APP_PORT="8080" \
-e API_KEY=<"your-API-KEY"> \
radinlawrence/ipgeolocation-api:v1
```
> Remember to replace " your-api-key " with your own key. Here I'm using the free plan of ipgeolocation.io to generate api_key: https://app.ipgeolocation.io/


**Front-end**

```bash
docker service create \
--name iplocation-frontend-service \
--network iplocation-net \
--replicas 4 \
--constraint 'node.labels.resource == disk' \
-e API_SERVER="iplocation-api-service" \
-e API_SERVER_PORT="8080" \
-e API_PATH="/api/v1/" \
-e APP_PORT="8080" \
-p 80:8080 \
radinlawrence/ipgeolocation-frontend:v1
```
![image](https://user-images.githubusercontent.com/100775027/166469990-7f8fd3ee-4ca6-4117-8df3-3c73e8ea91cd.png)

```bash
docekr service ls
```
![image](https://user-images.githubusercontent.com/100775027/166470803-b64ee551-fc2b-4f96-87ab-ac527b955c1d.png)



Now our setup is complete and you can point the frontend server public IP  or  can point the resource disk  labeled public IP server to a domain name.
eg: http://example.com/ip/8.8.8.8

![image](https://user-images.githubusercontent.com/100775027/166469649-2bb5a2f2-938c-4fbe-b619-2b26ee1f9516.png)

## Conclusion
This is how Set up IP geolocation using docker-swarm. Please contact me when you encounter any difficulty error while using this terrform code. Thank you and have a great day!


 ### ⚙️ Connect with Me
<p align="center">
<a href="https://www.linkedin.com/in/radin-lawrence-8b3270102/"><img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white"/></a>
<a href="mailto:radin.lawrence@gmail.com"><img src="https://img.shields.io/badge/Gmail-D14836?style=for-the-badge&logo=gmail&logoColor=white"/></a>

