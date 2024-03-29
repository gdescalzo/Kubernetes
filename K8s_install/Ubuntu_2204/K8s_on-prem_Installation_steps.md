
# Preparing the Enviroment for Kubernetes

> The purpose of this document it´s just **install K8s on-prem**, others documents, not this one, will have day two configurations.

## Disable Swap

> Doit on the Masters and Workers

```
sudo swapoff -a
vi /etc/fstab # and comment the SWAP line
```

## Enable Netfilter for ContainerD

> Doit on the Masters and Workers

```
sudo printf "overlay\nbr_netfilter\n" >> /etc/modules-load.d/containerd.conf
sudo modprobe overlay
sudo modprobe br_netfilter
sudo printf "net.bridge.bridge-nf-call-iptables = 1\nnet.ipv4.ip_forward = 1\nnet.bridge.bridge-nf-call-ip6tables = 1\n" >> /etc/sysctl.d/99-kubernetes-cri.conf
sudo sysctl --system
```

## Kubernetes and ContainerD repo

> Doit on the Masters and Workers

```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
```

## Install ContainerD

> Doit on the Masters and Workers

```
sudo apt-cache search containerd
sudo apt install containerd -y
sudo mkdir /etc/containerd
sudo sh -c "containerd config default > /etc/containerd/config.toml"
sudo sed -i 's/ SystemdCgroup = false/ SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl restart containerd.service
sudo systemctl enable --now containerd
sudo systemctl start --now containerd
sudo systemctl status containerd
sudo systemctl restart containerd
```

## Prepare "overlay" and "netfilter" for Kubernetes

> Doit on the Masters and Workers

```
 cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
 overlay
 br_netfilter
 EOF

 cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
 net.bridge.bridge-nf-call-iptables  = 1
 net.bridge.bridge-nf-call-ip6tables = 1
 net.ipv4.ip_forward                 = 1
 EOF

 sudo sysctl --system

```

<br />
<hr>

# Install Kubernetes

> Doit on the Masters and Workers

> Confirm if the required package are on the repo, the repo are constantly changing the URL.
>
> ```
> apt-cache search kubeadm && apt-cache search kubelet && apt-cache search kubectl
> ```

> Install K8s
>
> ```
>  sudo apt install kubeadm kubelet kubectl -y
> ```

> Mark the K8s pakage that don´t be upgraded on the next 'apt upgrade'

> ```
> sudo apt-mark hold kubeadm kubelet kubectl
> sudo systemctl enable kubelet.service
> sudo reboot
> sudo systemctl status kubelet.service
> sudo systemctl restart kubelet.service
> ```

> For K8s wokrs needs the configuration to start (do it on Master only)
> ```
> mkdir -p $HOME/.kube
> sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
> sudo chown $(id -u):$(id -g) $HOME/.kube/config
> ```


## Install Calico

```
 kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml

 curl https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/custom-resources.yaml -O

 sed -i 's/cidr: 192\.168\.0\.0\/16/cidr: 10.10.0.0\/16/g' custom-resources.yaml
```