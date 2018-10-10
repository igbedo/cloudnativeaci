### Need for Container Orchestration System?
To keep this simple, imagine that you had to run hundreds of containers. You can easily see that if they are running in a distributed mode, there are multiple features that you will need from a management angle to make sure that the cluster is up and running, is healthy and more.

Some of these necessary features include:

Health Checks on the Containers

Launching a fixed set of Containers for a particular Docker image

Scaling the number of Containers up and down depending on the load

Performing rolling update of software across containers

and more…

Let us look at how we can do some of that using Docker Swarm. The Docker Documentation and tutorial for trying out Swarm mode has been excellent.

### Master node
Note down the IP Address of the master node (your first Ip address)

```
ssh -i k8slab.pem ubuntu@master-ip

```
### Swarm Cluster
Now that our machines are setup, we can proceed with setting up the Swarm.

```
$ docker swarm init --advertise-addr MANAGER_IP

```
It looks like this:

```
docker@manager1:~$ docker swarm init — advertise-addr 192.168.1.8
Swarm initialized: current node (5oof62fetd4gry7o09jd9e0kf) is now a manager.
To add a worker to this swarm, run the following command:
docker swarm join \
 — token SWMTKN-1–5mgyf6ehuc5pfbmar00njd3oxv8nmjhteejaald3yzbef7osl1-ad7b1k8k3bl3aa3k3q13zivqd \
 192.168.1.8:2377
To add a manager to this swarm, run ‘docker swarm join-token manager’ and follow the instructions.
docker@manager1:~$

```
Great!

You will also notice that the output mentions the docker swarm join command to use in case you want another node to join as a worker. Keep in mind that you can have a node join as a worker or as a manager. At any point in time, there is only one LEADER and the other manager nodes will be as backup in case the current LEADER opts out.

At this point you can see your Swarm status by firing the following command as shown below:

```
docker@manager1:~$ docker node ls
ID              HOSTNAME STATUS AVAILABILITY MANAGER STATUS
5oof62fetd..*   manager1 Ready  Active       Leader

```

This shows that there is a single node so far i.e. manager1 and it has the value of Leader for the MANAGER column.

Stay in the SSH session itself for manager1.


### Joining as Worker Node

To find out what docker swarm command to use to join as a node, you will need to use the join-token <role> command.

To find out the join command for a worker, fire the following command:

```
docker@manager1:~$ docker swarm join-token worker

```
To add a worker to this swarm, run the following command:

```
docker swarm join \
 — token SWMTKN-1–5mgyf6ehuc5pfbmar00njd3oxv8nmjhteejaald3yzbef7osl1-ad7b1k8k3bl3aa3k3q13zivqd \
 192.168.1.8:2377

docker@manager1:~$
```
### Joining as Manager Node
To find out the the join command for a manager, fire the following command:

```
docker@manager1:~$ docker swarm join-token manager
To add a manager to this swarm, run the following command:
docker swarm join \
 — token SWMTKN-1–5mgyf6ehuc5pfbmar00njd3oxv8nmjhteejaald3yzbef7osl1–8xo0cmd6bryjrsh6w7op4enos \
 192.168.1.8:2377

docker@manager1:~$
```
Notice in both the above cases, that you are provided a token and it is joining the Manager node (you will be able to identify that the IP address is the same the MANAGER_IP address).

Keep the SSH to manager1 open. And fire up other command terminals for working with other worker docker machines.

### Adding Worker Nodes to our Swarm
Now that we know how to check the command to join as a worker, we can use that to do a SSH into each of the worker Docker machines and then fire the respective join command in them.

```
docker@manager1:~$ docker node ls
docker@manager1:~$

```
You should see 3 nodes, one as the manager (manager1) and the other 2 as workers.

Run the command

```
$ docker info 
```
and zoom into the Swarm section to check out the details for our Swarm.
```
Swarm: active
 NodeID: 5oof62fetd4gry7o09jd9e0kf
 Is Manager: true
 ClusterID: 6z3sqr1aqank2uimyzijzapz3
 Managers: 1
 Nodes: 2
 Orchestration:
  Task History Retention Limit: 5
 Raft:
  Snapshot Interval: 10000
  Heartbeat Tick: 1
  Election Tick: 3
 Dispatcher:
  Heartbeat Period: 5 seconds
 CA Configuration:
  Expiry Duration: 3 months
 Node Address: 192.168.1.8

```
### Notice a few of the properties:

The Swarm is marked as active. It has 3 Nodes in total and 1 manager among them.

Since I am running the docker info command on the manager1 itself, it shows the Is Manager as true.

The Raft section is the Raft consensus algorithm that is used. 

### Create a Service
Now that we have our swarm up and running, it is time to schedule our containers on it. This is the whole beauty of the orchestration layer. We are going to focus on the app and not worry about where the application is going to run.

All we are going to do is tell the manager to run the containers for us and it will take care of scheduling out the containers, sending the commands to the nodes and distributing it.

To start a service, you would need to have the following:

What is the Docker image that you want to run. In our case, we will run the standard nginx image that is officially available from the Docker hub.

We will expose our service on port 80.

We can specify the number of containers (or instances) to launch. This is specified via the replicas parameter.

We will decide on the name for our service. And keep that handy.

What I am going to do then is to launch 5 replicas of the nginx container. To do that, I am again in the SSH session for my manager1 node. And I give the following docker service create command:

```
docker service create --replicas 5 -p 80:80 --name web nginx
ctolq1t4h2o859t69j9pptyye
```
What has happened is that the Orchestration layer has now got to work.

You can find out the status of the service, by giving the following command:
```
docker@manager1:~$ docker service ls
ID            NAME  REPLICAS  IMAGE  COMMAND
ctolq1t4h2o8  web   0/5       nginx
```

This shows that the replicas are not yet ready. You will need to give that command a few times.

In the meanwhile, you can also see the status of the service and how it is getting orchestrated to the different nodes by using the following command:

```
docker@manager1:~$ docker service ps web
ID  NAME   IMAGE  NODE      DESIRED STATE  CURRENT STATE      ERROR
7i*  web.1  nginx  worker1   Running        Preparing 2 minutes ago
17*  web.2  nginx  manager1  Running        Running 22 seconds ago
ey*  web.3  nginx  worker2   Running        Running 2 minutes ago
bd*  web.4  nginx  worker1   Running        Running 45 seconds ago
dw*  web.5  nginx  worker2   Running        Running 2 minutes ago
```

This shows that the nodes are getting setup. It could take a while.

But notice a few things. In the list of nodes above, you can see that the 5 containers are being scheduled by the orchestration layer on manager1, worker1, worker2. 

A few executions of docker service ls shows the following responses:

```
docker@manager1:~$ docker service ls
ID            NAME  REPLICAS  IMAGE  COMMAND
ctolq1t4h2o8  web   3/5       nginx

docker@manager1:~$
```
and then finally:
```
docker@manager1:~$ docker service ls
ID              NAME REPLICAS IMAGE COMMAND
ctolq1t4h2o8    web  5/5      nginx
docker@manager1:~$
```
If we look at the service processes at this point, we can see the following:
```
docker@manager1:~$ docker service ps web
ID  NAME   IMAGE  NODE      DESIRED STATE  CURRENT STATE      ERROR
7i*  web.1  nginx  worker1   Running        Running 4 minutes ago
17*  web.2  nginx  manager1  Running        Running 7 minutes ago
ey*  web.3  nginx  worker2   Running        Running 9 minutes ago
bd*  web.4  nginx  worker2   Running        Running 8 minutes ago
dw*  web.5  nginx  manager1   Running        Running 9 minutes ago
```

If you do a docker ps on the manager1 node right now, you will find that the nginx daemon has been launched.
```
docker@manager1:~$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
933309b04630        nginx:latest        "nginx -g 'daemon off"   2 minutes ago       Up 2 minutes        80/tcp, 443/tcp     web.2.17d502y6qjhd1wqjle13nmjvc
docker@manager1:~$
```

### Accessing the Service
You can access the service by hitting any of the manager or worker nodes. It does not matter if the particular node does not have a container scheduled on it. That is the whole idea of the swarm.

Try out a curl to any of the Docker Machine IPs (manager1 or worker1/2) or hit the URL (http://<machine-ip>) in the browser. You should be able to get the standard NGINX Home page.


Ideally you would put the Docker Swarm service behind a Load Balancer.

### Scaling up and Scaling down
This is done via the docker service scale command. We currently have 5 containers running. Let us bump it up to 8 as shown below by executing the command on the manager1 node.

```
$ docker service scale web=8
web scaled to 8
```
Now, we can check the status of the service and the process tasks via the same commands as shown below:

```
docker@manager1:~$ docker service ls
ID           NAME REPLICAS IMAGE COMMAND
ctolq1t4h2o8 web  5/8      nginx
```

In the ps web command below, you will find that it has decided to schedule the new containers on worker1 (2 of them) and manager1(one of them)
```
docker@manager1:~$ docker service ps web
ID   NAME   IMAGE  NODE      DESIRED STATE  CURRENT STATE                 ERROR
7i*  web.1  nginx  worker1   Running    Running 14 minutes ago
17*  web.2  nginx  manager1  Running    Running 17 minutes ago
ey*  web.3  nginx  worker2   Running    Running 19 minutes ago
bd*  web.4  nginx  worker1   Running    Running 17 minutes ago
dw*  web.5  nginx  worker2   Running    Running 19 minutes ago
8t*  web.6  nginx  worker2   Running    Starting about a minute ago
b8*  web.7  nginx  manager1  Running    Ready less than a second ago
0k*  web.8  nginx  worker1   Running    Starting about a minute ago
```
We wait for a while and then everything looks good as shown below:
```
docker@manager1:~$ docker service ls
ID           NAME REPLICAS IMAGE COMMAND
ctolq1t4h2o8 web 8/8 nginx
docker@manager1:~$ docker service ps web
ID  NAME  IMAGE NODE     DESIRED STATE CURRENT STATE ERROR
7i* web.1 nginx worker3  Running       Running 16 minutes ago
17* web.2 nginx manager1 Running       Running 19 minutes ago
ey* web.3 nginx worker2  Running       Running 21 minutes ago
bd* web.4 nginx worker5  Running       Running 20 minutes ago
dw* web.5 nginx worker4  Running       Running 21 minutes ago
8t* web.6 nginx worker1  Running       Running 4 minutes ago
b8* web.7 nginx manager1 Running       Running 2 minutes ago
0k* web.8 nginx worker1  Running       Running 3 minutes ago
docker@manager1:~$
```
### Inspecting nodes
You can inspect the nodes anytime via the docker node inspect command.

For example if you are already on the node (for example manager1) that you want to check, you can use the name self for the node.
```
$ docker node inspect self
```
Or if you want to check up on the other nodes, give the node name. For e.g.
```
$ docker node inspect worker1
```

We can also use the docker node inspect command to check the availability of the node and as expected, you will find a section in the output as follows:
```
$ docker node inspect worker1
…..
"Spec": {
 "Role": "worker",
 "Availability": "active"
 },
…
or

docker@manager1:~$ docker node inspect — pretty worker1
ID: 9yq4lcmfg0382p39euk8lj9p4
Hostname: worker1
Joined at: 2016–09–16 08:32:24.5448505 +0000 utc
Status:
 State: Ready
 Availability: Active
Platform:
 Operating System: linux
 Architecture: x86_64
Resources:
 CPUs: 1
 Memory: 987.2 MiB
Plugins:
 Network: bridge, host, null, overlay
 Volume: local
Engine Version: 1.12.1
Engine Labels:
 — provider = hypervdocker@manager1:~$
We can see that it is “Active” for its Availability attribute.
```

### Remove the Service
You can simply use the service rm command as shown below:
```
docker@manager1:~$ docker service rm web
web
docker@manager1:~$ docker service ls
ID NAME REPLICAS IMAGE COMMAND
docker@manager1:~$ docker service inspect web
[]
Error: no such service: web
```
### Applying Rolling Updates
This is straight forward. In case you have an updated Docker image to roll out to the nodes, all you need to do is fire an service update command.

For e.g.
```
$ docker service update --image <imagename>:<version> web
```
