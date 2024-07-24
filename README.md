# Hello Every One Pls Like and Subscribe to Our Channel
 # https://youtu.be/g9Kv83C1St8


# Table of Contents


# Prerequisites
  # Lab Setup

Step 1 Set Hostname and Update Hosts file
Step 2 Disable Swap Space on Each Node
Step 3 Adjust SELinux and Firewall Rules for Kubernetes
Step 4 Add Kernel Modules and Parameters
Step 5 Install Conatinerd Runtime
Step 6 Install Kubernetes tools
Step 7 Install Kubernetes Cluster on Rocky Linux 9 / Alma Linux 9
Step 8 Install Calico Network Addon
Step 9 Test Kubernetes Cluster Installation


Prerequisites
A fresh Installation of Rocky Linux 9 or AlmaLinux 9
Sudo user with admin rights
Minimum of 2 GB RAM, 2 vCPUs and 20 GB Disk Space
A reliable Internet Connection

Lab Setup
We have used three Virtual machines with following specification.


192.168.254.167 kubecontroller.com
192.168.254.168 kube-node4.com
192.168.254.169 kube-node4.com
192.168.254.170 kube-node4.com
192.168.254.171 kube-node4.com



Sysops as sudo user on each node
Without any further delay, lets deep dive into Kubernetes installation steps.

Step 1: Set Hostname and Update Hosts file


#command
yum update -y 
yum install vim tar wget make unzip -y

#command

sudo hostnamectl set-hostname “kubecontroller.com” && exec bash
sudo hostnamectl set-hostname “kube-node1.com” && exec bash
sudo hostnamectl set-hostname “kube-node2.com” && exec bash
sudo hostnamectl set-hostname “kube-node3.com” && exec bash
sudo hostnamectl set-hostname “kube-node4.com” && exec bash


Add the following entries in vim /etc/hosts file on each node.

#command
vim /etc/hosts

192.168.254.167 kubecontroller.com  kubecontroller
192.168.254.168 kube-node4.com  kube-node1
192.168.254.169 kube-node4.com  kube-node2
192.168.254.170 kube-node4.com  kube-node4
192.168.254.171 kube-node4.com  kube-node4

:wq (save & exit)

#### To Ping the Servers #######

ping -c 5 192.168.254.167
ping -c 5 kubecontroller.com
ping -c 5 192.168.254.168
ping -c 5 kube-node4.com
ping -c 5 192.168.254.169
ping -c 5 kube-node4.com
ping -c 5 192.168.254.170
ping -c 5 kube-node4.com
ping -c 5 192.168.254.171
ping -c 5 kube-node4.com


#command   in all servers 

sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
sudo setenforce 0
sudo sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=permissive/g' /etc/sysconfig/selinux


--- In Master / Controller Server ------
## Command##

sudo firewall-cmd --permanent --add-port={6443,2379,2380,10250,10251,10252,10257,10259,179}/tcp
sudo firewall-cmd --permanent --add-port=4789/udp
sudo firewall-cmd --reload

It has to come 
Success
Success
Success


On the Worker Nodes, allow beneath ports in the firewall,

#command
sudo firewall-cmd --permanent --add-port={179,10250,30000-32767}/tcp
sudo firewall-cmd --permanent --add-port=4789/udp
sudo firewall-cmd --reload

It has to come 
Success
Success
Success

#command  in all servers 

sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF


#command in all servers 

sudo modprobe overlay
sudo modprobe br_netfilter


#command in all servers 

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf

net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF


#command in all servers 

sudo sysctl --system
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf install containerd.io -y



#command in all servers 

containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
sudo systemctl status containerd

#command in all servers 

cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF

#command#command in all servers 

sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
sudo systemctl enable --now kubelet
sudo systemctl status kubelet




-----------------------------------------------------------------------------
#command  in only Controller Server

sudo kubeadm init --control-plane-endpoint=kubecontroller.com




#output:



Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
export KUBECONFIG=/etc/kubernetes/admin.conf
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml

kubectl get nodes 
kubectl get pods 

Alternatively, if you are the root user, you can run:

export KUBECONFIG=/etc/kubernetes/admin.conf

kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

  kubeadm join k8s-master01:6443 --token 20ab12.ey9crmg7xsctftfd \
        --discovery-token-ca-cert-hash sha256:d43d23fc848617883f2d195c571a8a550d20653f09e2d730caaf3e6e5cff3d13 \
        --control-plane

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join k8s-master01:6443 --token 20ab12.ey9crmg7xsctftfd \
        --discovery-token-ca-cert-hash sha256:d43d23fc848617883f2d195c571a8a550d20653f09e2d730caaf3e6e5cff3d13
[root@k8s-master01 ~]#


Once above command is executed successfully, we will get following output,

Install-kubernetes-cluster-rockylinux9-almalinux9-kubeadm-command

From the output above make a note of the command which will be executed on the worker nodes to join the Kubernetes cluster.


To start interacting with Kubernetes cluster, run the following commands on the master node.

#command
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config


Next, join the worker nodes to the cluster, run following Kubeadm command from the worker nodes.

#command
kubeadm join k8s-master01:6443 --token 20ab12.ey9crmg7xsctftfd \
        --discovery-token-ca-cert-hash sha256:d43d23fc848617883f2d195c571a8a550d20653f09e2d730caaf3e6e5cff3d13

Output from Worker01

Worker01-Join-Kubernetes-Cluster

Output from Worker02

Worker02-Join-Kubernetes-Cluster

Now, head back to master node and run kubectl command to verify the nodes status.

#command
kubectl get nodes

Kubectl-Get-Nodes-RockyLinux-AlmaLinux

Output above shows that nodes is “NoteRead”, so to make the nodes status “Ready”, install Calico network addon or plugin in the next step.

Step 8: Install Calico Network Addon
Calico network addon is required on Kubernetes cluster to enable communication between pods, to make DNS service function with the cluster and to make the nodes status as Ready.

In order to install calico CNI (Container Network Interface) addon, run following kubectl commands from the master node only.


#command
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml

Install-Calico-Addon-Kubernetes-RockyLinux-AlmaLinux

Verify calico pods status,

#command
kubectl get pods -n kube-system

Calico-Pods-Status-Kubernetes-RockyLinux-AlmaLinux

Next, verify the nodes status, this time nodes status should be in Ready State.

#command
kubectl get nodes

Nodes-Status-Post-Calico-Addon-Installation

Perfect, output above confirms nodes are in Ready state and can handle workload. Let’s test our Kubernetes installation the next step.

Step 9: Test Kubernetes Cluster Installation
To test Kubernetes cluster installation, let’s try to deploy nginx based application using deployment. Run following kubectl commands,


In Docker Engine 

docker pull nagarjunadeepak/cloudzomato:latest

#command
kubectl create deployment zomato --image thanish/cloudzomato --replicas 5

kubectl expose deployment zomato --type NodePort --port 3000

kubectl get deployment zomato

kubectl get pods

kubectl get svc zomato

Test-Kubernetes-Installation-RockyLinux-AlmaLinux

Try to access the application using nodeport “31121”, run following curl command,

#command
curl k8s-worker02:30491

Access-Nginx-App-Kubernetes-RockyLinux-AlmaLinux

Then copy the Worker01 ip & port no. 
Ex: 192.168.254.170:31121/zomata

--------------------------------------------------
kubernetes integration with jenkins::
---------------------------------------------
kubectl create namespace jenkins1 
kubectl get ns 
kubect create sa jenkins1 -n jenkins1
kubectl create token jenkins1 -n jenkins1 --duration=87600h
kubectl create rolebinding jenkins-admin-binding --clusterrole=admin --serviceaccount jenkins1:jenkins1 --namespace=jenkins1
