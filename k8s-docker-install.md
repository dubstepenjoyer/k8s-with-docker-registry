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