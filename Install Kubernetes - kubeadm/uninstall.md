
## Upgrade Steps:

### Uninstall Kubernetes

-f option is to suppress the prompt and do a force reset, can be useful if you want to automate the process

```shell
sudo kubeadm reset -f
yum remove -y kubeadm kubectl kubelet kubernetes-cni kube*
sudo rm -rf /etc/cni /etc/kubernetes /var/lib/dockershim /var/lib/etcd /var/lib/kubelet /var/run/kubernetes ~/.kube/*

cat <<EOF | sudo tee k8s-uninstall.sh
#!/bin/sh
set -x
kubeadm reset --force
yum remove -y kubeadm kubectl kubelet kubernetes-cni kube*
yum autoremove -y
[ -e ~/.kube ] && rm -rf ~/.kube
[ -e /etc/kubernetes ] && rm -rf /etc/kubernetes
[ -e /opt/cni ] && rm -rf /opt/cni
EOF
chmod +x k8s-uninstall.sh
./k8s-uninstall.sh



chmod +x k8s-uninstall.sh 
./k8s-uninstall.sh
```
```shell
## Uninstall Docker
sudo yum remove docker-ce docker-ce-cli containerd.io -y
rm -rf /var/lib/docker

systemctl reset-failed
systemctl daemon-reload

## Remove  Mirantis docker cri-docker.service
rm -rf /usr/lib/systemd/system/cri-docker.service
rm -rf /usr/lib/systemd/system/cri-docker.socket

```
