# dncs-swarm-management
Repository for the second project of the DNCS course 2019/2020, themed as managing docker swarms.

# Preamble
For this project, we will simulate a network of IoT devices controlled by an MQTT broker that act as a docker swarm, and we will use swarmpit to monitor the swarm resource utilization.
In order to do so, we will start with FIWARE architecture as a base. They in fact provide some dummy IoT sensor useful for testing this architecture.

Ref: https://github.com/FIWARE/tutorials.IoT-over-MQTT

Tools needed:
- Docker 19.03.5+
- Docker Machine 0.13.0+

# First step
The first step is that of creating a number of virtual machine so as to divide the components of our swarm, since it's a requirement for this project. We choose to create 3 nodes for our swarm, but that can be limited to 2 or exanded with many more depending on the characteristics of your machine.

The following commands inside the docker-machine-init.sh script will do this for you:
```bash
docker-machine create node1
docker-machine create node2
docker-machine create node3
```

# Second step, setting up the swarm
Once that was done, we can ssh into node1:
```bash
docker-machine ssh node1
docker-machine ssh node3
```
Once done we need to set up the swarm, so we'll do it inside node1:
```bash
docker swarm init --advertise-addr 192.168.99.100
```
This way, node1 has been set in swarm mode and set up as a manager/leader of the swarm. The address published will be accessible from our browser and from the other nodes once added to the swarm.

Now we need to add node2 and node3 to the swarm.
Node2:
```bash
docker-machine ssh node2
``` 
Once inside node2, we need to copy the docker swarm join command with its token given by node1 inside node2, for example (but it will be a different token on your machine):
```bash
docker swarm join --token SWMTKN-1-2t9k3101x61dh0b618lh04i8siva4tflvh8mfv72vqho6zd4ol-7o4x93pbgyg17pfwdapogid8o 192.168.99.100:2377
```
The same goes for node3:
```bash
docker-machine ssh node3
docker swarm join --token SWMTKN-1-2t9k3101x61dh0b618lh04i8siva4tflvh8mfv72vqho6zd4ol-7o4x93pbgyg17pfwdapogid8o 192.168.99.100:2377
```
As a check for our node cluster we can run:
```bash
docker node ls
```
To see that all the three nodes have been created and are up and running.
Now, if we want, we can see that we are able to ping the various nodes of the swarm from each other, so congrats, now we have a fully working 3-node swarm!

# Third step, running the docker stack deploy script
Now, we could do everything manually, but it will be cumbersome, so the ideal is to deploy everything using a .yaml script, in the same fashion as a docker-compose.yaml script, but for swarms.

Go ahead and copy this github repo in your node1 with the command:
```bash
git clone https://github.com/mstreet97/dncs-swarm-management.git
```
Now we can finally deploy our stack with:
```bash
docker stack deploy -c docker-compose.yaml fiware-swarm
```
After having done this, we can check if everything is up and running by giving the command:
```bash
docker stack ps fiware-stack
```
Which will answer showing all the containers up and running and explicitly declaring on which node they are.

Next we need to run the provisioning script, that provisions all the sensor in the four stores with:
```bash
bash provisioner.sh
```
Once this has been done, we can head to:
```bash
192.168.99.100:3000/device/monitor
```
To be able to interact with the dummy IoT devices and see the MQTT message exchange.
NOTE: Every ip of the three node stack will do it, so we can even use:
```bash
192.168.99.101:3000/device/monitor
192.168.99.102:3000/device/monitor
```
and obtain the same result.

Still missing to do:
- fixing the actual swarm deploy, since it's only been tested successfully locally. There is a problem with the provisioning file, which the cURL returns an internal server error and honestly I have no idea why
- swarm monitoring with swarmpit