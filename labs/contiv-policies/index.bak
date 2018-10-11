## Contiv Policies With Docker containers

Contiv is an Open source project driven primarily by Cisco for policy based networking, storage and cluster management for containerized applications. 

### Container Policy
Policies have become critical to control the business logic in a Cloud environment. There are 2 ways to describe policy. In imperative model, policy is defined in terms of how the end goal is achieved. For example, we specify the filters and actions with Openflow protocol that achieves end goal of packet handling and this is an example of imperative model. In declarative model, policy is defined in terms of the end goal and it gives flexibility to the end-system to implement the policy in different ways. Congress and Opflex are examples of declarative policy model. With declarative model, it is possible to specify the policy in terms of business logic without specifying implementation detail. For example, the business logic can say that web container should not talk to database container. The implementation of this business logic can be achieved by having an iptables rule or by having a hardware tcam rule to block specific ports. In a cloud computing world, policies can be defined for compute, storage and networking. Both Containers and VM needs policies to implement business logic. Following are examples of some policies that can be applied to applications deployed in Cloud using either VMs or Containers:

 * Authorization policy – Specifies tenants and their privileges.
 * Resource usage policy – Specifies resource constraints for tenants, containers and VMs.
 * Application access policy – Specifies containers that can communicate to each other and containers that are exposed to outside world.

## Contiv Networking
Contiv Networking project provides policy based networking for Docker Containers. Following are some details on Contiv Networking:


* There are two primary components of Contiv: Contiv netplugin is implemented as a Docker networking plugin and sits on every Docker host. Contiv netmaster runs centrally as a cluster which consumes the policy and provides REST api to the outside world.

* Contiv netplugin can also work with Container schedulers like Kubernetes, Swarm, Mesos.
* Openvswitch is used to implement network policy with Openflow based pipeline. This allows for Contiv to have a flexible policy as well as a performance oriented solution.
* Networking can be provided using pure L3, pure L2 or using Overlays based on the underlying network.
* Policies are specified in terms of business logic using right abstractions rather than using Networking language.
* Multi-tenant support is available.
* Contiv provides integration with Cisco ACI fabric. This allows for ACI fabric to implement Contiv policy. This is still in preliminary stages.
* Service discovery is integrated.
* “netctl” is the CLI frontend provided by Contiv to interact with Contiv master.
* “contiv-compose” can be used to automate policy deployment. This is still in the early stages.

### Contiv Networking policy
Contiv networking policy is specified in terms of business logic rather than Networking constraints. The policy is targeted towards clear division of responsibilities between Development and Operation teams.  Following picture from Contiv website illustrates the Contiv Networking policy model:

![picture alt](https://github.com/igbedo/cloudnativeaci/blob/master/labs/contiv1.png "Net policy Diagram")

Following are some notes on the Contiv Networking policy:

* The top level object is Tenant. Tenant can be a customer inside a cloud network or different groups within an single organization. Networks, policies and applications are defined under a Tenant.

* End point group(EPG) allows for grouping a set of applications which needs to have similar policies. For example, all web containers that are exposed to outside world need to open up “http” and “https” port. “Web” EPG can be defined to encapsulate all web containers and policy can be applied to “Web” EPG rather than applying the policy to individual Web containers. EPG is a Cisco construct that is being used also in Cisco ACI as well as in Openstack Group based policy.

* Workflow looks like this:
   * Create Tenant, Network.
   * Create Endpoint groups for container groupings.
   * Create policy between Endpoint groups.
   * Create application containers and tie them to the Endpoint group(EPG), Tenant and Network.

Contiv Networking policy can be specified using “netctl” tool or by a JSON file that is consumed by “contiv-compose” tool. “contiv-compose” tool is built on top of libcompose. Compose file has similar syntax as docker-compose YAML file.

Following is an example of Contiv networking policy “p1” that contains 3 rules to block all incoming tcp ports other than port 80 and 443 from EPG “c1”.

```
netctl policy create p1
netctl policy rule-add p1 1 -g c1 -n test -direction=in -protocol=tcp -action=deny
netctl policy rule-add p1 2 -g c1 -n test -direction=in -protocol=tcp -port=80 -action=allow -priority=10
netctl policy rule-add p1 3 -g c1 -n test -direction=in -protocol=tcp -port=443 -action=allow -priority=10
```

Following is a sample policy specified as JSON file. The policy template language is still a work in progress within Contiv project. In the policy below, we define one tenant “io.contiv.tenant”. One user “vagrant” is defined who has access to “test” and “prod” networks. The user “vagrant” also has access to three policies “TrustApp”, “WebDefault”, “Websecure”. “TrustApp” policy trusts all ports exposed by the application. “WebDefault” exposes both “http” and “https” ports. “WebSecure” exposes only “https” port.
```
{
	"LabelMap`" : {
		"Tenant" : "io.contiv.tenant",
		"NetworkIsolationPolicy" : "io.contiv.policy"
	},

	"UserPolicy" : [

		{ "User":"vagrant", 
		  "DefaultTenant": "default",
		  "Networks": "test,prod",
		  "DefaultNetwork": "prod",
		  "NetworkPolicies" : "TrustApp,WebDefault,WebSecure",
		  "DefaultNetworkPolicy": "TrustApp" }
	],

	"NetworkPolicy" : [
		{ "Name":"AllPriviliges", 
		  "Rules": ["permit all"]},

		{ "Name":"WebDefault", 
		  "Rules": ["permit tcp/80", "permit tcp/443"] },

		{ "Name":"WebSecure", 
		  "Rules": ["permit tcp/443"] },
		  
		{ "Name":"TrustApp",
		  "Rules": ["permit app"] }
	]
}

```

## Creating Policies

Let's implement the following  business Policy details for an application:


* Clients can talk to webserver using secure(http) and unsecure(https) methods in “test” network.
* Clients can talk to webserver using only secure method(https) in “prod” network.
* This application is run in default tenant network.

Following picture shows the Contiv object model for this application:

![picture alt](https://github.com/igbedo/cloudnativeaci/blob/master/labs/contiv21.png "Net policy Diagram")

Following commands creates this application using “netctl” CLI:

```
# Network
netctl net create test --subnet=10.1.1.0/24
netctl net create prod --subnet=20.1.1.0/24

# Create policy
netctl policy create p1
netctl policy rule-add p1 1 -g c1 -n test -direction=in -protocol=tcp -action=deny
netctl policy rule-add p1 2 -g c1 -n test -direction=in -protocol=tcp -port=80 -action=allow -priority=10
netctl policy rule-add p1 3 -g c1 -n test -direction=in -protocol=tcp -port=443 -action=allow -priority=10

netctl policy create p2
netctl policy rule-add p2 4 -g c2 -n prod -direction=in -protocol=tcp -action=deny
netctl policy rule-add p2 5 -g c2 -n prod -direction=in -protocol=tcp -port=443 -action=allow -priority=10

# Create group and associate with network and policy
netctl group create test c1 
netctl group create test web1 -policy=p1
netctl group create prod c2
netctl group create prod web2 -policy=p2

# Create services
docker run -d --net web1.test --name web1_1 --dns 10.1.1.2 nginx
docker run -d --net web1.test --name web1_2 --dns 10.1.1.2 nginx
docker run -ti --net c1.test --name c1_1 --dns 10.1.1.2 smakam/myubuntu:v3

docker run -d --net web2.prod --name web2_1 --dns 20.1.1.2 nginx
docker run -d --net web2.prod --name web2_2 --dns 20.1.1.2 nginx
docker run -ti --net c2.prod --name c2_1 --dns 20.1.1.2 smakam/myubuntu:v3 bash

```

Note: The docker image “smakam/myubuntu:v3” is inherited from “ubuntu” Container  has network tools installed for testing.

Let’s login to client Container “c1_1” and check that it is able to reach service “web1.test.default”. Following output shows that the ping request is getting load balanced between “web1_1.test.default” and “web1_2.test.default”.
```
# ping -c1 web1.test.default
PING web1.test.default (10.1.1.5) 56(84) bytes of data.
64 bytes from 10.1.1.5: icmp_seq=1 ttl=64 time=0.120 ms

--- web1.test.default ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.120/0.120/0.120/0.000 ms
# ping -c1 web1.test.default
PING web1.test.default (10.1.1.6) 56(84) bytes of data.
64 bytes from 10.1.1.6: icmp_seq=1 ttl=64 time=6.13 ms

--- web1.test.default ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 6.137/6.137/6.137/0.000 ms
```

To check that the policy is working, lets login to “c1_1” service and check that only port 80 and 443 is open in “web1” service as specified by policy “p1”.
```
# nc -zvw 1 web1.test.default 79-81
nc: connect to web1.test.default port 79 (tcp) timed out: Operation now in progress
nc: connect to web1.test.default port 79 (tcp) timed out: Operation now in progress
Connection to web1.test.default 80 port [tcp/http] succeeded!
nc: connect to web1.test.default port 81 (tcp) timed out: Operation now in progress
nc: connect to web1.test.default port 81 (tcp) timed out: Operation now in progress
# nc -zvw 1 web1.test.default 442-444
nc: connect to web1.test.default port 442 (tcp) timed out: Operation now in progress
nc: connect to web1.test.default port 442 (tcp) timed out: Operation now in progress
nc: connect to web1.test.default port 443 (tcp) failed: Connection refused
nc: connect to web1.test.default port 443 (tcp) failed: Connection refused
nc: connect to web1.test.default port 444 (tcp) timed out: Operation now in progress
nc: connect to web1.test.default port 444 (tcp) timed out: Operation now in progress
```
As we can see above, only port 80 and 443 is open. We are getting connection refused for port 443 because we have not enabled the authentication scheme for https.

To check that the policy is working, lets login to “c2_1” and check that only port 443 is open in “web2” service as specified by policy “p2”.
```
root@78871d2b3159:/# nc -zvw 1 web2.prod.default 79-81
nc: connect to web2.prod.default port 79 (tcp) timed out: Operation now in progress
nc: connect to web2.prod.default port 79 (tcp) timed out: Operation now in progress
nc: connect to web2.prod.default port 80 (tcp) timed out: Operation now in progress
nc: connect to web2.prod.default port 80 (tcp) timed out: Operation now in progress
nc: connect to web2.prod.default port 81 (tcp) timed out: Operation now in progress
nc: connect to web2.prod.default port 81 (tcp) timed out: Operation now in progress
root@78871d2b3159:/# nc -zvw 1 web2.prod.default 442-444
nc: connect to web2.prod.default port 442 (tcp) timed out: Operation now in progress
nc: connect to web2.prod.default port 442 (tcp) timed out: Operation now in progress
nc: connect to web2.prod.default port 443 (tcp) failed: Connection refused
nc: connect to web2.prod.default port 443 (tcp) failed: Connection refused
nc: connect to web2.prod.default port 444 (tcp) timed out: Operation now in progress
nc: connect to web2.prod.default port 444 (tcp) timed out: Operation now in progress
```
Let’s look at tenants, networks, epgs, policies created using netctl:
```
$ netctl tenant ls
Name     
------   
default  

$ netctl network list
Tenant   Network  Nw Type  Encap type  Packet tag  Subnet        Gateway
------   -------  -------  ----------  ----------  -------       ------
default  test     data     vxlan       0           10.11.1.0/24  
default  prod     data     vxlan       0           10.11.2.0/24 

$ netctl group ls
Tenant   Group        Network  Policies
------   -----        -------  --------
default  test_client  test     
default  test_web     test     test_web-in
default  prod_client  prod     
default  prod_web     prod     prod_web-in

$ netctl policy list
Tenant   Policy
------   ------
default  test_web-in
default  prod_web-in

$ netctl policy rule-ls prod_web-in
Incoming Rules:
Rule  Priority  From EndpointGroup  From Network  From IpAddress  Protocol  Port  Action
----  --------  ------------------  ------------  ---------       --------  ----  ------
1     1                             prod          10.11.2.0/24    tcp       0     deny
2     2         prod_client         prod                          tcp       443   allow

$ netctl policy rule-ls test_web-in
Incoming Rules:
Rule  Priority  From EndpointGroup  From Network  From IpAddress  Protocol  Port  Action
----  --------  ------------------  ------------  ---------       --------  ----  ------
1     1                             test          10.11.1.0/24    tcp       0     deny
2     2         test_client         test                          tcp       80    allow
3     3         test_client         test                          tcp       443   allow
```
To test that the policy is working, we can try the following commands in client container and it should show that only port 80, 443 is exposed in “test” network and port 443 is exposed in “prod” network.
```
nc -zvw 1 prod_web.prod.default 79-81
nc -zvw 1 prod_web.prod.default 442-444

nc -zvw 1 test_web.test.default 79-81
nc -zvw 1 test_web.test.default 442-444
```
