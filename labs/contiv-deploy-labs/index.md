## Deploy Kubernetes with Contiv CNI

Contiv can be deployed manually through the following steps:

#### Step 0: Assumes Docker  have been deployed
```
$ssh -i k8slab.pem ubuntu@ip-of-master-node
$sudo su -

apt-get  update  &&  apt-get  install  -y  apt-transport-https

curl  -s https://packages.cloud.google.com/apt/doc/apt-key.gpg |  apt-key  add  -

cat  <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial  main
EOF

apt-get  update
apt-get  install  -y  kubelet=1.11.3-00  kubeadm=1.11.3-00  kubectl

```
#### Step 1: Initialize Kubernetes with IP for Contiv

```
kubeadm init --kubernetes-version=1.11.3 --service-cidr 10.254.0.0/16

```
Wait for the output from the init process. The end of the meessage  will look lik this:

```
mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  You can now join any number of machines by running the following on each node
as root:

  kubeadm join 172.31.19.85:6443 --token ymbt8i.z3swtn1p03l26lbs --discovery-token-ca-cert-hash sha256:da310ad42c8c9d0dda6b3c036830d01c5898a517238f173c04a0fdfd054ed12a

```
#### 3. Download Contiv package

```
curl -L -O https://github.com/contiv/install/releases/download/1.1.7/contiv-1.1.7.tgz
tar xvzf contiv-1.1.7.tgz 
cd contiv-1.1.7/
```
Install and Run Contiv using as parameter the private IP of the machine on which us running

```
./install/k8s/install.sh -n <MACHINE_PRIVATE_IP>

serviceaccount/contiv-netmaster unchanged
configmap/contiv-config unchanged
daemonset.extensions/contiv-netplugin created
replicaset.extensions/contiv-netmaster configured
daemonset.extensions/contiv-etcd unchanged
Installation is complete
=========================================================

Contiv UI is available at https://18.144.15.234:10000
Please use the first run wizard or configure the setup as follows:
 Configure forwarding mode (optional, default is routing).
 netctl global set --fwd-mode routing
 Configure ACI mode (optional)
 netctl global set --fabric-mode aci --vlan-range <start>-<end>
 Create a default network
 netctl net create -t default --subnet=<CIDR> default-net
 For example, netctl net create -t default --subnet=20.1.1.0/24 -g 20.1.1.1 default-net

=========================================================
 
 root@ip-172-31-19-85:~/contiv-1.1.7# kubectl get pods --all-namespaces
NAMESPACE     NAME                                      READY   STATUS              RESTARTS   AGE
kube-system   contiv-etcd-ph2rj                         1/1     Running             0          3m
kube-system   contiv-netmaster-qw4v6                    3/3     Running             0          3m
kube-system   contiv-netplugin-m2x2d                    2/2     Running             0          2m
kube-system   coredns-78fcdf6894-fjtmb                  0/1     ContainerCreating   0          5m
kube-system   coredns-78fcdf6894-frd76                  0/1     ContainerCreating   0          5m
kube-system   etcd-ip-172-31-19-85                      1/1     Running             0          4m
kube-system   kube-apiserver-ip-172-31-19-85            1/1     Running             0          5m
kube-system   kube-controller-manager-ip-172-31-19-85   1/1     Running             0          4m
kube-system   kube-proxy-5hgjf                          1/1     Running             0          5m
kube-system   kube-scheduler-ip-172-31-19-85            1/1     Running             0          5m

``` 
#### 4. Provide minimal configuration for Contiv, which includes the overlay network address
 COPY AND PASTE FROM THE OUTPUT OF PREVIOUS

```
 netctl net create -t default --subnet=20.1.1.0/24 -g 20.1.1.1 default-net

root@ip-172-31-19-85:~/contiv-1.1.7# kubectl get pods --all-namespaces
NAMESPACE     NAME                                      READY   STATUS    RESTARTS   AGE
kube-system   contiv-etcd-ph2rj                         1/1     Running   0          5m
kube-system   contiv-netmaster-qw4v6                    3/3     Running   0          5m
kube-system   contiv-netplugin-m2x2d                    2/2     Running   0          4m
kube-system   coredns-78fcdf6894-fjtmb                  1/1     Running   0          7m
kube-system   coredns-78fcdf6894-frd76                  1/1     Running   0          7m
kube-system   etcd-ip-172-31-19-85                      1/1     Running   0          6m
kube-system   kube-apiserver-ip-172-31-19-85            1/1     Running   0          7m
kube-system   kube-controller-manager-ip-172-31-19-85   1/1     Running   0          6m
kube-system   kube-proxy-5hgjf                          1/1     Running   0          7m
kube-system   kube-scheduler-ip-172-31-19-85            1/1     Running   0          7m
root@ip-172-31-19-85:~/contiv-1.1.7# 

```
#### 5. Access Contive dashboard at the MACHINE-PUBLIC-IP of the machine
https://public-ip:10000
login as admin/admin

#### 6. Test by deploying a deployment object
```
kubectl run nginx --image-nginx

root@ip-172-31-20-129:~/contiv-1.1.7# kubectl get pods 
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
default       nginx-64f497f8fd-xx54w                     1/1     Running   0          33m

```


## Adding Nodes to the Cluster
#### Step 0: Assumes Docker  have been deployed

```
ssh -i k8slab.pem ubuntu@ip-address-of-worker-node
sudo su -

apt-get  update  &&  apt-get  install  -y  apt-transport-https

curl  -s https://packages.cloud.google.com/apt/doc/apt-key.gpg |  apt-key  add  -

cat  <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial  main
EOF

apt-get  update
apt-get  install  -y  kubelet=1.11.3-00  kubeadm=1.11.3-00  kubectl

```
#### Step : Copy the join string from the output of Master init code

```
kubeadm join 172.31.19.85:6443 --token ymbt8i.z3swtn1p03l26lbs --discovery-token-ca-cert-hash sha256:da310ad42c8c9d0dda6b3c036830d01c5898a517238f173c04a0fdfd054ed12a
```
#### Step 2: Run this from the master Node

```
root@ip-172-31-19-85:~/contiv-1.1.7# kubectl get pods --all-namespaces
NAMESPACE     NAME                                      READY   STATUS              RESTARTS   AGE
default       nginx-64f497f8fd-sp5qp                    0/1     Pending             0          5m
kube-system   contiv-etcd-ph2rj                         1/1     Running             0          15m
kube-system   contiv-netmaster-qw4v6                    3/3     Running             0          15m
kube-system   contiv-netplugin-jctdt                    0/2     ContainerCreating   0          24s
kube-system   contiv-netplugin-m2x2d                    2/2     Running             0          14m
kube-system   coredns-78fcdf6894-fjtmb                  1/1     Running             0          17m
kube-system   coredns-78fcdf6894-frd76                  1/1     Running             0          17m
kube-system   etcd-ip-172-31-19-85                      1/1     Running             0          16m
kube-system   kube-apiserver-ip-172-31-19-85            1/1     Running             0          16m
kube-system   kube-controller-manager-ip-172-31-19-85   1/1     Running             0          16m
kube-system   kube-proxy-5hgjf                          1/1     Running             0          17m
kube-system   kube-proxy-kq6cv                          0/1     ContainerCreating   0          24s
kube-system   kube-scheduler-ip-172-31-19-85            1/1     Running             0          17m
```
#### As you add more nodes:
```
root@ip-172-31-19-85:~/contiv-1.1.7# kubectl get pods --all-namespaces
NAMESPACE     NAME                                      READY   STATUS    RESTARTS   AGE
default       nginx-64f497f8fd-sp5qp                    1/1     Running   0          13m
kube-system   contiv-etcd-ph2rj                         1/1     Running   0          22m
kube-system   contiv-netmaster-qw4v6                    3/3     Running   0          22m
kube-system   contiv-netplugin-bdppd                    2/2     Running   0          34s
kube-system   contiv-netplugin-jctdt                    2/2     Running   0          8m
kube-system   contiv-netplugin-m2x2d                    2/2     Running   0          22m
kube-system   coredns-78fcdf6894-fjtmb                  1/1     Running   0          25m
kube-system   coredns-78fcdf6894-frd76                  1/1     Running   0          25m
kube-system   etcd-ip-172-31-19-85                      1/1     Running   0          24m
kube-system   kube-apiserver-ip-172-31-19-85            1/1     Running   0          24m
kube-system   kube-controller-manager-ip-172-31-19-85   1/1     Running   0          24m
kube-system   kube-proxy-5hgjf                          1/1     Running   0          25m
kube-system   kube-proxy-kq6cv                          1/1     Running   0          8m
kube-system   kube-proxy-v8kth                          1/1     Running   0          34s
kube-system   kube-scheduler-ip-172-31-19-85            1/1     Running   0          24m

```

