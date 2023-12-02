# Guide how to install k8s cluster and connect it with docker registry using https

## Firstly you need to install k8s and docker on your machines. Then start up local docker registry with ssl certificate. Follow steps below to install k8s and docker on your node.

### Set hostname

```
sudo hostnamectl set-hostname master-node
```
or 
```
sudo hostnamectl set-hostname worker-node
```
ATTENTION! hostname must be unique, i.e. master1-node or master-node1 (2, 3, 4 ...) and same for worker!

### Add hostname into ```/etc/hosts```

On worker node(s)
```
echo "1.2.3.4 master-node" | sudo tee -a /etc/hosts
```
On master node(s)
```
echo "1.2.3.5 worker-node" | sudo tee -a /etc/hosts
```

i.e. your master's node ```/etc/hosts``` should contain access for worker node and and vice versa. 

### Update hostnamed service

```
sudo systemctl restart systemd-hostnamed
```
and
```
exec bash
```

### Turn-off swap file

```
sudo swapoff -a
```

It is necessary to commit the changes, that when rebooting the swap file was not included

```
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

### Download kernel modules

```
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
```

```
sudo modprobe overlay
```

```
sudo modprobe br_netfilter
```

Install kernel parameters

```
sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
```

Apply changes

```
sudo sysctl --system
```

### Add Docker official key

```
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
```

### Add Docker repository

```
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

```
sudo apt update
```

### Install packeges

```
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates containerd.io
```

```
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
```

```
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
```

```
sudo systemctl restart containerd
```

```
sudo systemctl enable containerd
```

### Connection k8s repository

```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```

```
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```

```
sudo apt update
```

### Install k8s tools

```
sudo apt install -y kubelet kubeadm kubectl
```

```
sudo apt-mark hold kubelet kubeadm kubectl
```

# END OF WORKER INSTALLATION

## Run below commands on your master-node to make it master

```
sudo kubeadm init --control-plane-endpoint=master-node
```

```
mkdir -p $HOME/.kube
```

```
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```

```
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Run this to see a command that you need to run on your worker node to add it.

```
kubeadm token create --print-join-command
```

## Add Calico

```
curl https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml -O
```

## Install Calico

```
kubectl apply -f calico.yaml
```

# Addition

## Docker installation

```
sudo apt update
```

```
sudo apt install ca-certificates gnupg
```

```
sudo install -m 0755 -d /etc/apt/keyrings
```

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

```
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```
sudo apt update
```

```
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## To avoid some issues with docker permissions do [this](https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user)

```
sudo groupadd docker
```

```
sudo usermod -aG docker $USER
```

```
newgrp docker
```

# I used this [guide](https://www.heyvaldemar.net/ustanovka-kubernetes-na-ubuntu-server-22-04-lts/) and [docker documentation](https://docs.docker.com/engine/install/ubuntu/)

# Now go to ```./certs``` and read ```.md``` file