# KubernetesClusterSetup

### Install Docker - Master and All Worker Nodes

### Docker Engine version 20.10.7
To install Docker Engine, see Install Docker Engine .
Docker Compose version 1.29.2
To install Docker Compose, see Install Docker Compose  

1. On all nodes - master and worker 1 - 6
```
yum list docker-ce --showduplicates | sort -r
yum install docker-ce-20.10.17 docker-ce-cli-20.10.17 containerd.io -y
systemctl start docker
systemctl enable docker
```

```Installing docker-compose
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```


### Disable SELinux, Firewalld and swap - Master and All Worker Nodes

#### Update sysctl settings for Kubernetes networking
```cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```
#### Disable SELinux
```
setenforce 0
sed -i --follow-symlinks 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/sysconfig/selinux
#### Disable Firewall
systemctl disable firewalld
systemctl stop firewalld
```

#### Disable swap
```sed -i '/swap/d' /etc/fstab
swapoff -a
#### on every node exec
cat <<EOF | sudo tee /etc/docker/daemon.json
{
"exec-opts": ["native.cgroupdriver=systemd"],
"log-driver": "json-file",
"log-opts": {
"max-size": "100m"
},
"storage-driver": "overlay2"
}
EOF
systemctl daemon-reload
systemctl restart docker
```

### Setup cri-dockerd from Mirantis

#### Install Go
```
wget https://go.dev/dl/go1.20.7.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.20.7.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
go version
```


#### Follow the document -
*** https://github.com/Mirantis/cri-dockerd#using-cri-dockerd ***

#### Clone the cri-docker repo 
```
git clone https://github.com/Mirantis/cri-dockerd.git
cd cri-dockerd/
make cri-dockerd
install -o root -g root -m 0755 cri-dockerd /usr/local/bin/cri-dockerd
install packaging/systemd/* /etc/systemd/system
sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
systemctl daemon-reload
systemctl enable --now cri-docker.socket
```

### Kubernetes Setup - Master and All Worker Nodes
```
cat >>/etc/yum.repos.d/kubernetes.repo<<EOF
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
yum install -y kubelet-1.25.11 kubeadm-1.25.11 kubectl-1.25.11 --disableexcludes=kubernetes
systemctl enable --now kubelet
```


### On the Master Node
```
kubeadm init --apiserver-advertise-address=192.168.100.20 --pod-network-cidr=10.244.0.0/16 --cri-socket=/var/run/cri-dockerd.sock

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.100.20:6443 --token nqp1ik.d6imfmtitr3bkhri \
        --discovery-token-ca-cert-hash sha256:0438bad56ade0cec40aea5faacf70e015ba4d4c8089f009b6d5c19a04a5cfca9
```
### On the Worker Nodes - 
```
kubeadm join 192.168.100.20:6443 --token nqp1ik.d6imfmtitr3bkhri --discovery-token-ca-cert-hash sha256:0438bad56ade0cec40aea5faacf70e015ba4d4c8089f009b6d5c19a04a5cfca9 --cri-socket=/var/run/cri-dockerd.sock
```
