
Simple example, to create, install and test Kubernetes Cluster locally with a behavior similar to a classic K8s cluster
To do it we need the tools below
 - Vagrant
 - VirtualBox
 - Ubuntu or Gnu/Linux distribution


## Kubernetes Cluster
 Install kubernetes cluster by using 2 linux virtuals machines.

 We use Vagrant to create and configure our vms
 You can find in this directory the *Vagrantfile*

 ### 1. disable swap (root) :
 ```sh   
    swapoff -a
    vim /etc/fstab (comment swap line)
    mount -a
```

### 2. Install packages :
```sh
    apt-get update && apt-get install -y apt-transport-https curl
```

### 3. Add google apt repository (root)
```sh    
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
    add-apt-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```

### 4. install kubernetes binaries:
```sh    
     apt-get install -y kubelet kubeadm kubectl kubernetes-cni
     systemctl enable kubelet
```
- kubeadm : cluster installation
- kubelet : an agent that runs on each node (machine)
- kubectl : allow communication with the cluster

### 5. initilization of the master (kubmaster)
update cgroup driver for kubelet
```sh
# kubmaster & kubnode
vim /etc/default/kubelet
KUBELET_EXTRA_ARGS="--cgroup-driver=cgroupfs"
```

```sh
   kubeadm init --apiserver-advertise-address=192.168.56.101 --node-name $HOSTNAME 
    --pod-network-cidr=10.244.0.0/16
```
command above will : 
- define IP that the ip will use
- define the name of the node
- define the range of pods network

Note: if an error occurend please use this command to reset 
```sh
   kubeadm reset
```
Create the config file after the init 

```sh
 mkdir -p $HOME/.kube
 sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
 sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
### 6. establishment of the intern network 
```sh
    # (kubmaster & kubnode)
    sysctl net.bridge.bridge-nf-call-iptables=1 

    # (kubmaster)
    kubectl apply -f 
    https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

    # if necessary (kubmaster)
    kubectl edit cm -n kube-system kube-flannel-cfg
    # edit network 10.244.0.0/16 to 10.10.0.0/16 pour dashboard
```

### 7. check system's pods status 
```sh
  # (kubmaster)
  kubectl get pod --all-namespaces
  kubectl get pods
```
### 8. join the node
```sh
    # on the kubemaster to get token
    kubeadm token list

    # join the node (kubnode)
    kubeadm join 192.168.56.101:6443 --token b6tkd1.8w1355qi09mjhr7x \
        --discovery-token-ca-cert-hash sha256:5132023d4d10607a3ab096c91cb12e1d00f5b9f93310f7b82d385c4325840fe4 

    # check nodes (kubmaster)
    kubectl get nodes
    # you will see 2 nodes (master et node)
```
### Note : fix IP issue
 if you are use like me Vagrant/Virtualbox + Flannel pods, perhaps you will have the same Ip
 issue, to check this:
```sh
    # (kubmaster) 
    kubectl get nodes -o wide
```
if you get the same "INTERNAL-IP" addr for kubmaster and kubenode (like 10.0.2.15),
 it's NOT NORMAL !!
To fix that: 
 - Update the /etc/hosts in the kubmaster add this line: 192.168.56.101  kubmaster kubmaster
 - Update the /etc/hosts in the kubnode, add this line : 192.168.56.102  kubnode kubnode
 - Delete flannel pods
 to get kube system pods:
```sh
# (kubmaster) 
kubectl get pods -n kube-system
```
we delete flannel pods (2 pods)
```sh
# (kubmaster) 
kubectl delete pods kube-flannel-<XXXXXXX> -n kube-system
```
now Kubernetes will regenerates these two pods, and they will got the correct ips
you can check by  ```kubectl get nodes -o wide```

### 9. auto completion and alias
first install bach completion package on ubuntu
```sh
apt install bash-completion
```
to enable kubectl autocompletion

```sh
echo 'source <(kubectl completion bash)' >>~/.bashrc
```
we define some aliases for using kubernetes 

```sh
# .bash_aliases
alias k='kubectl'
alias kcc='kubectl config current-context'
alias kg='kubectl get'
alias kga='kubectl get all --all-namespaces'
alias kgp='kubectl get pods'
alias kgs='kubectl get services'
alias kgpk='kubectl get pods -n kube-system'
alias kcuc='kubectl config use-context'
```
and reload bashrc ``` source .bashrc ```

### 10. remote access to K8s cluster
- Install kubectm in your os 
- Copy the k8s token in your local .kube/config
example
```sh
# in your local console
ssh vagrant@kubmaster "sudo cat /etc/kubernetes/admin.conf" >.kube/config
```