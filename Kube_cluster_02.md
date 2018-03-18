# Setup Kubernetes cluster in Debian 9

## Basic software

- As root
```
cd /root
apt-get install -y vim mc uml-utilities ntp qemu-guest-agent \
htop sudo curl git git-core etckeeper zsh apt-transport-https ca-certificates \
bridge-utils gettext-base
usermod root -s /bin/zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
sed -ri 's/ZSH_THEME="robbyrussell"/ZSH_THEME="pygmalion"/g' .zshrc
sed -ri 's/plugins=\(git\)/plugins=\(debian apt systemd docker zsh-navigation-tools\)/g' .zshrc
echo 'export VTYSH_PAGER=more' >> /etc/zsh/zshenv
source .zshrc
```

## Install Kubernetes

-- With docker from repository

```
apt-get update && apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF > /etc/apt/sources.list.d/kubernetes.list  
deb http://apt.kubernetes.io/ kubernetes-xenial main  
EOF
apt-get update && apt-get install -y docker.io kubelet kubeadm kubectl kubernetes-cni
```

-- Adding NEW repo to install docker-ce

```
apt-get update
apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
add-apt-repository \
   "deb https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
   $(lsb_release -cs) \
   stable"
apt-get update && apt-get install -y docker-ce=$(apt-cache madison docker-ce | grep 17.03 | head -1 | awk '{print $3}')
apt-get update && apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
```

### Basic initialization of cluster

``` Run only in Kube Master
kubeadm init
```

### Initialization with a specific pod network (Flannel)

```
sysctl net.bridge.bridge-nf-call-iptables="1"
kubeadm init --pod-network-cidr=10.244.0.0/16
```

------
- Message after initialization:

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a /**regular user**/:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now /**deploy a pod network**/ to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node /**as root**/:

  kubeadm join --token 0f538e.ff7741bc11169121 IP_OF_K8S_MASTER:6443 --discovery-token-ca-cert-hash sha256:_supertoken_for_adding_nodes_to_k8s_cluster_

- End of initialization message
----

### Deploying pod network

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/kube-flannel.yml
kubectl get pods --all-namespaces
```
- Check that Kube-DNS is running

### Join pods

- Install tools

```
---> See "Install Kubernetes" and install with the same method as master:
 - With docker from repository
 - Adding NEW repo to install docker-ce

```

- Join pod to cluster (/**as root**/)

```
  kubeadm join --token 0f538e.ff7741bc11169121 IP_OF_K8S_MASTER:6443 --discovery-token-ca-cert-hash sha256:_supertoken_for_adding_nodes_to_k8s_cluster_
```

- Check STATUS of cluster in the Kube Master

```
kubectl get nodes
```

Enjoy!

## Links

```
- Kubeadm reference: https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm/
- Using Kubeadm to Create a cluster:
https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/
- Components:
https://kubernetes.io/docs/setup/independent/install-kubeadm/
- Pod network:
https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/
```
