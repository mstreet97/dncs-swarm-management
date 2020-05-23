# Docker Swarm Management and Statistics Project
Repository for the second project of the Design of Networks and Communication Systems course @UniTn, Academic Year 2019/2020, themed as managing docker swarms.
Authors: Matteo Strada, Giovanni Costa.
The considered scenario is that of two groups of IoT sensors, one that updates frequently (sensor in a brewery cask) and one that updates much more slowly (the weather station). The performance and statistics of the swarm and the various nodes and services will be monitored via swarmpit.io

# Preamble
For this project, we will simulate a network of IoT devices controlled by an MQTT broker that act as a docker swarm, and we will use swarmpit to monitor the swarm resource utilization.
In order to do so, we will create different dummy IoT sensors, which basically are only the publisher part of the eclipse-mosquitto mqtt clients, that, as a standalone docker image or combined with other (e.g installed on a ubuntu image) will send data to the actual mqtt broker once every determined period. Another image, which is the subscriber of eclipse-mosquitto, listens for messages on that topic and will be used to collect them and logs them.

Tools needed:
- Docker 19.03.5+
- Docker Machine 0.13.0+

Because of the above requirements, please make sure that Docker (https://docs.docker.com/get-docker/) is installed and up to date for your machine. (Mandatory)
Please make sure that also docker-machine is installed and up to date (https://docs.docker.com/machine/). (This step is optional, but highly recommended since it's the easier way to follow this guide, but anyway you can use whatever virtualization tool that might suit your needs, like Digital Ocean, but you might need to adapt this tutorial content on your own if you want to go that way.)

# First step
The first step is that of creating a number of virtual machines so as to divide the components of our swarm, since it's a requirement for this project. We choose to create 3 nodes for our swarm, but that can be limited to 2 or expanded with many more depending on the characteristics of your machine.

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
Now, if we want, we can see that we are able to ping the various nodes of the swarm from each other, so congrats, we have a fully working 3-node swarm!

# Third step, running the docker stack deploy script
Now, we could do everything manually, but it will be cumbersome, so the ideal is to deploy everything using a .yaml script, in the same fashion as a docker-compose.yaml script, but for swarms.

Go ahead and copy this github repo in your node1 with the command:
```bash
git clone https://github.com/mstreet97/dncs-swarm-management.git
```
Now we can finally deploy our stack with:
```bash
cd dncs-swarm-management
docker stack deploy -c docker-compose.yaml mqtt-swarm
```
Alternatively you can also get only the docker-compose.yaml file using wget and directly running that, since all the needed scripts (e.g. sunriseSunset.js are automatically retrieved by the remote repo once the docker service is created).

After having done this, we can check if everything is up and running by giving the command:
```bash
docker stack ps mqtt-swarm
```
Which will answer showing all the containers up and running and explicitly declaring on which node they are.

Now, if we give the command:
```bash
watch docker service log mqtt-swarm_subscriber
```
It will automatically run that command every 2 seconds, so that we can see the logs expanding as more mqtt messages are received.

After docker is done downloading all the needed images and starting the services, we can log into swarmpit gui via a web page with either node ip at port 888:
```bash
192.168.99.100:888
```
After choosing an admin username and password we have access to swarmpit dashboard which shows the usage statistics for our stack and let us perform different tasks, including moving containers around, restarting or redeploying them and viewing their logs.

To see the MQTT publisher and subscribers interacting we can go to: services > weatherSubscriber > view log. Here the full message exchange will be shown.
This project assumed a "slow updating iot service", our dummy weather station which will show something like this clocking every 5 minutes:
```bash 
| Location: Trento,IT
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | Coordinates: Lat: 46.066423, Lng: 11.12576
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | Sunrise time: 6:13
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | Sunset time: 16:45
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | Weather report: Trento
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | 
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    |     \  /       Partly cloudy
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    |   _ /"".-.     -1 °C          
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    |     \_(   ).   ↓ 0 km/h       
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    |     /(___(__)  10 km          
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    |                0.0 mm         
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    |                                                        ┌─────────────┐                                                       
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | ┌──────────────────────────────┬───────────────────────┤  Tue 18 Feb ├───────────────────────┬──────────────────────────────┐
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | │            Morning           │             Noon      └──────┬──────┘     Evening           │             Night            │
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | ├──────────────────────────────┼──────────────────────────────┼──────────────────────────────┼──────────────────────────────┤
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | │      .-.      Light drizzle  │    \  /       Partly cloudy  │  _`/"".-.     Patchy rain po…│  _`/"".-.     Patchy rain po…│
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | │     (   ).    5..6 °C        │  _ /"".-.     8..9 °C        │   ,\_(   ).   6 °C           │   ,\_(   ).   2 °C           │
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | │    (___(__)   ↓ 6-8 km/h     │    \_(   ).   ↓ 5-6 km/h     │    /(___(__)  ↖ 2-3 km/h     │    /(___(__)  ↗ 1-2 km/h     │
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | │     ‘ ‘ ‘ ‘   7 km           │    /(___(__)  10 km          │      ‘ ‘ ‘ ‘  10 km          │      ‘ ‘ ‘ ‘  10 km          │
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | │    ‘ ‘ ‘ ‘    0.1 mm | 28%   │               0.0 mm | 0%    │     ‘ ‘ ‘ ‘   0.1 mm | 83%   │     ‘ ‘ ‘ ‘   0.1 mm | 27%   │
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | └──────────────────────────────┴──────────────────────────────┴──────────────────────────────┴──────────────────────────────┘
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    |                                                        ┌─────────────┐                                                       
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | ┌──────────────────────────────┬───────────────────────┤  Wed 19 Feb ├───────────────────────┬──────────────────────────────┐
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | │            Morning           │             Noon      └──────┬──────┘     Evening           │             Night            │
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | ├──────────────────────────────┼──────────────────────────────┼──────────────────────────────┼──────────────────────────────┤
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | │      .-.      Patchy light r…│      .-.      Patchy light r…│  _`/"".-.     Moderate rain …│  _`/"".-.     Patchy rain po…│
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | │     (   ).    0 °C           │     (   ).    2 °C           │   ,\_(   ).   -2..0 °C       │   ,\_(   ).   -6..-3 °C      │
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | │    (___(__)   ↖ 3-5 km/h     │    (___(__)   ↖ 3-4 km/h     │    /(___(__)  ← 5-10 km/h    │    /(___(__)  ↓ 7-15 km/h    │
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | │     ‘ ‘ ‘ ‘   9 km           │     ‘ ‘ ‘ ‘   9 km           │    ‚‘‚‘‚‘‚‘   9 km           │      ‘ ‘ ‘ ‘  10 km          │
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | │    ‘ ‘ ‘ ‘    1.0 mm | 29%   │    ‘ ‘ ‘ ‘    1.1 mm | 49%   │    ‚’‚’‚’‚’   0.8 mm | 24%   │     ‘ ‘ ‘ ‘   0.3 mm | 0%    │
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | └──────────────────────────────┴──────────────────────────────┴──────────────────────────────┴──────────────────────────────┘
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    |                                                        ┌─────────────┐                                                       
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | ┌──────────────────────────────┬───────────────────────┤  Thu 20 Feb ├───────────────────────┬──────────────────────────────┐
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | │            Morning           │             Noon      └──────┬──────┘     Evening           │             Night            │
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | ├──────────────────────────────┼──────────────────────────────┼──────────────────────────────┼──────────────────────────────┤
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | │    \  /       Partly cloudy  │    \  /       Partly cloudy  │    \  /       Partly cloudy  │    \  /       Partly cloudy  │
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | │  _ /"".-.     1 °C           │  _ /"".-.     2 °C           │  _ /"".-.     -1 °C          │  _ /"".-.     -3 °C          │
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | │    \_(   ).   ↖ 1-2 km/h     │    \_(   ).   ↗ 3-5 km/h     │    \_(   ).   ↑ 3-6 km/h     │    \_(   ).   ↑ 1-2 km/h     │
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | │    /(___(__)  10 km          │    /(___(__)  10 km          │    /(___(__)  10 km          │    /(___(__)  10 km          │
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | │               0.0 mm | 0%    │               0.0 mm | 0%    │               0.0 mm | 0%    │               0.0 mm | 0%    │
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | └──────────────────────────────┴──────────────────────────────┴──────────────────────────────┴──────────────────────────────┘
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | Location: Trento, Territorio Val Adige, TN, TAA, Italia [46.0664228,11.1257601]
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | 
mqtt-swarm_subscriber.1.uxz7t7nuat0b@node2    | Follow @igor_chubin for wttr.in updates
```

And also a "quickly updating iot service" (every 5 seconds) in the form of a brewery sensor ecosystem, with temperature, volume and pressure inside a beer cask. In order to see these logs, you should go with the swarmpit GUI to: services > brewerySubscriber > view log.
```bash
 Cask Volume: 5.05401359944gal
 
 Cask Temperature: 62°F
 
 Cask Pressure: 14.9033819006PSI
```

From the SwarmPit Gui a lot of different actions can be carried out, from monitoring nodes and container statuses, resources consumption, etc. We can (as was a request of the project) redeploy a service on a different container. The whole system by now should be working fine, feel free to experiment with you own containers and configurations.

# Final remarks
An ELK stack, to dynamically handle logging from the various containers is available, but it's been commented out in the docker-compose.yaml file. This is because it's quite resource heavy, and might take a long time to get it up and running, plus, specifically on my machine, 4 nodes are required to have it running and it's fairly unstable. If you have better hardware, feel free to have a go with it by uncommenting all the related lines.
Other that this, you will also need to set on each node an appropriate value for max_map_count.
Typing this command for each of the nodes of the docker swarm will be sufficient:
```bash
sudo sysctl -w vm.max_map_count=262144
```

