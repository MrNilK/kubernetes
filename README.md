# kubernetes

#Set up a Highly Available Kubernetes Cluster using kubeadm & HAProxy

Follow this documentation to set up a highly available Kubernetes cluster using Ubuntu 20.04 LTS.
This documentation guides you in setting up a cluster with two master nodes, two worker nodes and a load balancer node using HAProxy.

Environment
Role	FQDN	IP	OS	RAM	CPU	API Version
Master	kmaster1.example.com	10.0.1.24	Ubuntu 20.04	4G	2	1.19
Master	kmaster2.example.com	10.0.1.100	Ubuntu 20.04	4G	2	1.19
Worker	kworker1.example.com	10.0.1.162	Ubuntu 20.04	1G	1	1.19
Worker	kworker2.example.com	10.0.1.5	Ubuntu 20.04	1G	1	1.19
HAproxy	lb.example.com	10.0.1.84	Ubuntu 20.04	1G	1	
