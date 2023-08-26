# Set up a Highly Available Kubernetes Cluster using kubeadm and HAproxy
Follow this documentation to set up a highly available Kubernetes cluster using __Ubuntu 20.04 LTS__.

This documentation guides you in setting up a cluster with two master nodes, two worker nodes and a load balancer node using HAProxy.

## Environment
|Role|FQDN|IP|OS|RAM|CPU|API Version|
|----|----|----|----|----|----|----|
|Master|kmaster1.example.com|172.31.2.201|Ubuntu 20.04|4G|2|1.19|
|Master|kmaster2.example.com|172.31.3.131|Ubuntu 20.04|4G|2|1.19|
|Worker|kworker1.example.com|172.31.37.17|Ubuntu 20.04|1G|1|1.19|
|Worker|kworker2.example.com|172.31.36.174|Ubuntu 20.04|1G|1|1.19|
|HAproxy|lb.example.com|1172.31.45.53|Ubuntu 20.04|1G|1|


> * Perform all the commands as root user unless otherwise specified

## become a roor
```
sudo -i

```

## add entry in /etc/hosts in all machines
##### /etc/hosts
```
172.31.2.201 kmaster1.example.com example.com kmaster1
172.31.3.131 kmaster2.example.com example.com kmaster2
172.31.37.17 kworker1.example.com example.com kworker1
172.31.36.174 kworker2.example.com example.com kworker2
172.31.45.53 klb.example.com example.com klb

```

## Set up load balancer node
##### Install Haproxy
```
apt update && apt install -y haproxy
```
##### Configure haproxy
Append the below lines to **/etc/haproxy/haproxy.cfg**
```
frontend kubernetes-frontend
    bind 172.31.45.53:6443
    mode tcp
    option tcplog
    default_backend kubernetes-backend

backend kubernetes-backend
    mode tcp
    option tcp-check
    balance roundrobin
    server kmaster1 172.31.2.201:6443 check fall 3 rise 2
    server kmaster2 172.31.3.131:6443 check fall 3 rise 2
```
##### Restart haproxy service
```
systemctl restart haproxy
```

## On all kubernetes nodes (kmaster1, kmaster2, kworker1, kworker2)
##### Disable Firewall
```
ufw disable
```
##### Disable swap
```
swapoff -a; sed -i '/swap/d' /etc/fstab
```
##### Update sysctl settings for Kubernetes networking
```
cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```
##### Install docker engine
```
{
  apt install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
  add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
  apt update && apt install -y docker-ce=5:19.03.10~3-0~ubuntu-focal containerd.io
}
```
### Kubernetes Setup
##### Add Apt repository
```
{
  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
  echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
}
```
##### Install Kubernetes components
```
apt update && apt install -y kubeadm=1.19.2-00 kubelet=1.19.2-00 kubectl=1.19.2-00
```
## On any one of the Kubernetes master node (Eg: kmaster1)
##### Initialize Kubernetes Cluster
```
kubeadm init --control-plane-endpoint="172.31.45.53:6443" --upload-certs --apiserver-advertise-address=172.31.2.201 --pod-network-cidr=192.168.0.0/16
```
Copy the commands to join other master nodes and worker nodes.

```
the output ofkubeadm init command generate kubeadm join(other masters) , some directories and kubeadm join(worker only)
directories you need to create as a regular user, get out of root and become a regularuser
```
##### Deploy Calico network
```
kubectl --kubeconfig=/etc/kubernetes/admin.conf create -f https://docs.projectcalico.org/v3.15/manifests/calico.yaml
```

## Join other nodes to the cluster (kmaster2 & kworker1, kworker2)
> Use the respective kubeadm join commands you copied from the output of kubeadm init command on the first master.

> IMPORTANT: You also need to pass --apiserver-advertise-address to the join command when you join the other master node.

## Downloading kube config to your local machine
On your host machine
```
mkdir ~/.kube
scp root@10.0.1.24:/etc/kubernetes/admin.conf ~/.kube/config
```

## Verifying the cluster
```
kubectl cluster-info
kubectl get nodes
kubectl get cs
```

Have Fun!!
