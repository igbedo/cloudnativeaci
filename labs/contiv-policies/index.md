## Contiv Policies Intro 

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

Contiv Networking policy can be specified using “netctl” tool.
