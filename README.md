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



If you don’t have the command available, you can run the following command on a manager node to retrieve the join command for a worker:

```bash
docker swarm join-token worker
```

Run this generated join token in both of your worker instnaces:



To view information about nodes and verify the workers has been added to master:

```bash
dokcer node ls
```

## Drain manager node on the swarm

