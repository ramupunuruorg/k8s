# Title
This project is implemented to set up K8S cluster and intended to touch and feel of Micro Services architecture with containerized apps.
# Infrastructure Set Up
```
https://github.com/ramupunuruorg/aws-sandbox.git
```
# K8S Cluster setup using kubeadm tool
# Single node K8S cluster
* Step 1: Disable Swap, Add kernel Parameters and load kernal modules 
```
sudo swapoff -a && sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```
```
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
```
```
sudo modprobe overlay && sudo modprobe br_netfilter
```
```
sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
```
```
sudo sysctl --system && echo 1 > /proc/sys/net/ipv4/ip_forward
```
* Step 2: Install Docker
```
https://docs.docker.com/engine/install/debian/
```
* Step 3: CRI - Dockerd:Install Docker as runtime
```
VER=$(curl -s https://api.github.com/repos/Mirantis/cri-dockerd/releases/latest|grep tag_name | cut -d '"' -f 4|sed 's/v//g')
echo $VER
```
```
sudo mkdir /tmp/packages && cd /tmp/packages/
sudo apt install wget -y
sudo wget https://github.com/Mirantis/cri-dockerd/releases/download/v${VER}/cri-dockerd-${VER}.amd64.tgz
sudo tar xvf cri-dockerd-${VER}.amd64.tgz
```
```
cp cri-dockerd/cri-dockerd /usr/local/bin/
cri-dockerd --version
wget https://raw.githubusercontent.com/Mirantis/cri-dockerd/master/packaging/systemd/cri-docker.service
wget https://raw.githubusercontent.com/Mirantis/cri-dockerd/master/packaging/systemd/cri-docker.socket
sudo mv cri-docker.socket cri-docker.service /etc/systemd/system/
sudo sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
```
```
sudo systemctl daemon-reload
sudo systemctl enable cri-docker.service
sudo systemctl enable --now cri-docker.socket
sudo systemctl status cri-docker.socket
```
* Step 4: Add APT repository for kubernetes installation
```
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
* Step 5: install kubelet kubeadm kubectl on all nodes
```
sudo apt update -y
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
* Step 6: Initialize Kubernetes Cluster with Kubeadm tool
```
kubeadm init --cri-socket unix:///var/run/cri-dockerd.sock
```
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
* Step 7: Install Calico Network Plugin to enable networking on cluster
```
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml
```
* Step 8: To allow master node to be scheduled
```
kubectl get nodes
kubectl taint nodes master-node-1 node-role.kubernetes.io/control-plane:NoSchedule-
```  
