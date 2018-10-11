## Multi-host networking

There are many solutions like Contiv such as Calico, Weave, OpenShift, OpenContrail, Nuage, VMWare, Docker, Kubernetes, and OpenStack that provide solutions to multi-host container networking.

In this section, let's examine Contiv and Kubernetes overlay solutions.

### Multi-host networking with Contiv

Let's spin up two pods on the two different hosts.

1. Create a multi-host network

```
[vagrant@kubeadm-master ~]$ netctl net create --subnet=10.1.2.0/24 --gateway=10.1.2.1 contiv-net
Creating network default:contiv-net

[vagrant@kubeadm-master ~]$ netctl net ls
Tenant   Network      Nw Type  Encap type  Packet tag  Subnet        Gateway    IPv6Subnet  IPv6Gateway  Cfgd Tag
------   -------      -------  ----------  ----------  -------       ------     ----------  -----------  ---------
default  contivh1     infra    vxlan       0           132.1.1.0/24  132.1.1.1
default  default-net  data     vxlan       0           20.1.1.0/24   20.1.1.1
default  contiv-net   data     vxlan       0           10.1.2.0/24   10.1.2.1

[vagrant@kubeadm-master ~]$ netctl net inspect contiv-net
{
  "Config": {
    "key": "default:contiv-net",
    "encap": "vxlan",
    "gateway": "10.1.2.1",
    "networkName": "contiv-net",
    "nwType": "data",
    "subnet": "10.1.2.0/24",
    "tenantName": "default",
    "link-sets": {},
    "links": {
      "Tenant": {
        "type": "tenant",
        "key": "default"
      }
    }
  },
  "Oper": {
    "allocatedIPAddresses": "10.1.2.1",
    "availableIPAddresses": "10.1.2.2-10.1.2.254",
    "externalPktTag": 3,
    "networkTag": "contiv-net.default",
    "pktTag": 3
  }
}

```
We can now spin a couple of pods belonging to the contiv-net network. We will create one pod on the master node and one pod on the worker node. This will create a pod on the master node because we have specified tolerations in accordance with the master node's taints.

```
[vagrant@kubeadm-master ~]$ cat <<EOF > contiv-c1.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    io.contiv.tenant: default
    io.contiv.network: contiv-net
    k8s-app: contiv-c1
  name: contiv-c1
spec: 
  tolerations:
   - key: node-role.kubernetes.io/master
     effect: NoSchedule
  nodeSelector:
    node-role.kubernetes.io/master: ""
  containers: 
    - 
      image: contiv/alpine
      name: alpine
      command: 
      - sleep
      - "6000"
EOF

[vagrant@kubeadm-master ~]$ kubectl create -f contiv-c1.yaml
pod "contiv-c1" created

[vagrant@kubeadm-master ~]$ kubectl get pods -o wide
NAME                         READY     STATUS    RESTARTS   AGE       IP         NODE
contiv-c1                    1/1       Running   0          36s       10.1.2.2   kubeadm-master
vanilla-c-1408101207-p6swm   1/1       Running   1          5m        20.1.1.3   kubeadm-worker0

[vagrant@kubeadm-master ~]$ kubectl exec -it contiv-c1 sh
/ # ifconfig
eth0      Link encap:Ethernet  HWaddr 02:02:0A:01:02:02
          inet addr:10.1.2.2  Bcast:0.0.0.0  Mask:255.255.255.0
          inet6 addr: fe80::2:aff:fe01:202/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1450  Metric:1
          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
          TX packets:8 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:648 (648.0 B)  TX bytes:648 (648.0 B)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

/ # exit
```
The IP address of this pod is 10.1.2.2 which means it is part of the contiv-net network.

```
[vagrant@kubeadm-master ~]$ cat <<EOF > contiv-c2.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    io.contiv.tenant: default
    io.contiv.network: contiv-net
    k8s-app: contiv-c2
  name: contiv-c2
spec: 
  containers: 
    - 
      image: contiv/alpine
      name: alpine
      command: 
      - sleep
      - "6000"
EOF

[vagrant@kubeadm-master ~]$ kubectl create -f contiv-c2.yaml
pod "contiv-c2" created

[vagrant@kubeadm-master ~]$ kubectl get pods -o wide
NAME                        READY     STATUS    RESTARTS   AGE       IP         NODE
contiv-c1                   1/1       Running   0          27s       10.1.2.2   kubeadm-master
contiv-c2                   1/1       Running   0          6s        10.1.2.3   kubeadm-worker0
vanilla-c-357166617-s5tff   1/1       Running   1          1m        20.1.1.5   kubeadm-worker0

[vagrant@kubeadm-master ~]$ kubectl exec -it contiv-c2 sh
/ # ifconfig
eth0      Link encap:Ethernet  HWaddr 02:02:0A:01:02:03
          inet addr:10.1.2.3  Bcast:0.0.0.0  Mask:255.255.255.0
          inet6 addr: fe80::2:aff:fe01:203/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1450  Metric:1
          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
          TX packets:8 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:648 (648.0 B)  TX bytes:648 (648.0 B)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

/ # exit
```
Now let's try to ping between these two pods.
```
[vagrant@kubeadm-master ~]$ kubectl exec -it contiv-c1 sh
/ # ping -c 3 10.1.2.3
PING 10.1.2.3 (10.1.2.3): 56 data bytes
64 bytes from 10.1.2.3: seq=0 ttl=64 time=3.877 ms
64 bytes from 10.1.2.3: seq=1 ttl=64 time=1.213 ms
64 bytes from 10.1.2.3: seq=2 ttl=64 time=0.877 ms

--- 10.1.2.3 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.877/1.989/3.877 ms
/ # exit
```
## Using multiple tenants with arbitrary IPs in the networks

First, let's create a new tenant space.

```
[vagrant@kubeadm-master ~]$ netctl tenant create blue
Creating tenant: blue

[vagrant@kubeadm-master ~]$ netctl tenant ls
Name
------
blue
default
```
After the tenant is created, we can create network within tenant blue. Here we can choose the same subnet and network name as we used earlier with default tenant, as namespaces are isolated across tenants.

```
[vagrant@kubeadm-master ~]$ netctl net create -t blue --subnet=10.1.2.0/24  -g 10.1.2.1 contiv-net
Creating network blue:contiv-net

[vagrant@kubeadm-master ~]$ netctl net ls -t blue
Tenant  Network     Nw Type  Encap type  Packet tag  Subnet       Gateway   IPv6Subnet  IPv6Gateway  Cfgd Tag
------  -------     -------  ----------  ----------  -------      ------    ----------  -----------  ---------
blue    contiv-net  data     vxlan       0           10.1.2.0/24  10.1.2.1

```
Next, we can run pods belonging to this tenant.

```
[vagrant@kubeadm-master ~]$ cat <<EOF > contiv-blue-c1.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    io.contiv.tenant: blue
    io.contiv.network: contiv-net
    k8s-app: contiv-blue-c1
  name: contiv-blue-c1
spec: 

  containers: 
    - 
      image: contiv/alpine
      name: alpine
      command: 
      - sleep
      - "6000"
EOF

[vagrant@kubeadm-master ~]$ kubectl create -f contiv-blue-c1.yaml
pod "contiv-blue-c1" created

[vagrant@kubeadm-master ~]$ cat <<EOF > contiv-blue-c2.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    io.contiv.tenant: blue
    io.contiv.network: contiv-net
    k8s-app: contiv-blue-c2
  name: contiv-blue-c2
spec: 
  tolerations:
   - key: node-role.kubernetes.io/master
     effect: NoSchedule
  nodeSelector:
    node-role.kubernetes.io/master: ""
  containers: 
    - 
      image: contiv/alpine
      name: alpine
      command: 
      - sleep
      - "6000"
EOF

[vagrant@kubeadm-master ~]$ kubectl create -f contiv-blue-c2.yaml
pod "contiv-blue-c2" created

[vagrant@kubeadm-master ~]$ cat <<EOF > contiv-blue-c3.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    io.contiv.tenant: blue
    io.contiv.network: contiv-net
    k8s-app: contiv-blue-c3
  name: contiv-blue-c3
spec: 
  containers: 
    - 
      image: contiv/alpine
      name: alpine
      command: 
      - sleep
      - "6000"
EOF

[vagrant@kubeadm-master ~]$ kubectl create -f contiv-blue-c3.yaml
pod "contiv-blue-c3" created

```
Let's see what has been created.

```
[vagrant@kubeadm-master ~]$ kubectl get pods -o wide
NAME                         READY     STATUS    RESTARTS   AGE       IP         NODE
contiv-blue-c1               1/1       Running   0          53s       10.1.2.2   kubeadm-worker0
contiv-blue-c2               1/1       Running   0          20s       10.1.2.3   kubeadm-master
contiv-blue-c3               1/1       Running   0          6s        10.1.2.4   kubeadm-worker0
contiv-c1                    1/1       Running   0          6m        10.1.2.2   kubeadm-master
contiv-c2                    1/1       Running   0          4m        10.1.2.3   kubeadm-worker0
vanilla-c-1408101207-p6swm   1/1       Running   1          12m       20.1.1.3   kubeadm-worker0

Now, let's try to ping between these pods.

[vagrant@kubeadm-master ~]$ kubectl exec -it contiv-blue-c1 sh
/ # ping -c 3 10.1.2.3
PING 10.1.2.3 (10.1.2.3): 56 data bytes
64 bytes from 10.1.2.3: seq=0 ttl=64 time=11.651 ms
64 bytes from 10.1.2.3: seq=1 ttl=64 time=0.927 ms
64 bytes from 10.1.2.3: seq=2 ttl=64 time=0.890 ms

--- 10.1.2.3 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.890/4.489/11.651 ms
/ # ping -c 3 10.1.2.4
PING 10.1.2.4 (10.1.2.4): 56 data bytes
64 bytes from 10.1.2.4: seq=0 ttl=64 time=1.033 ms
64 bytes from 10.1.2.4: seq=1 ttl=64 time=0.086 ms
64 bytes from 10.1.2.4: seq=2 ttl=64 time=0.086 ms

--- 10.1.2.4 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.086/0.401/1.033 ms
/ # exit

```
## See the containers connectted to the network

Let's see which pods are on contiv-net.
```
[vagrant@kubeadm-master ~]$ netctl net inspect contiv-net
{
  "Config": {
    "key": "default:contiv-net",
    "encap": "vxlan",
    "gateway": "10.1.2.1",
    "networkName": "contiv-net",
    "nwType": "data",
    "subnet": "10.1.2.0/24",
    "tenantName": "default",
    "link-sets": {},
    "links": {
      "Tenant": {
        "type": "tenant",
        "key": "default"
      }
    }
  },
  "Oper": {
    "allocatedAddressesCount": 2,
    "allocatedIPAddresses": "10.1.2.1-10.1.2.3",
    "availableIPAddresses": "10.1.2.4-10.1.2.254",
    "endpoints": [
      {
        "containerName": "contiv-c1",
        "endpointID": "21f97ca11caa355fd56c75869b60587f149257932095c552fc3af76b0be0d91f",
        "homingHost": "kubeadm-master",
        "ipAddress": [
          "10.1.2.2",
          ""
        ],
        "labels": "map[]",
        "macAddress": "02:02:0a:01:02:02",
        "network": "contiv-net.default"
      },
      {
        "containerName": "contiv-c2",
        "endpointID": "a9506061759f9d36304997929ed66c360e09ae01aaa98b65394ae8c463cad158",
        "homingHost": "kubeadm-worker0",
        "ipAddress": [
          "10.1.2.3",
          ""
        ],
        "labels": "map[]",
        "macAddress": "02:02:0a:01:02:03",
        "network": "contiv-net.default"
      }
    ],
    "externalPktTag": 3,
    "networkTag": "contiv-net.default",
    "numEndpoints": 2,
    "pktTag": 3
  }
}
```
contiv-c1 and contiv-c2 are on contiv-net. 

