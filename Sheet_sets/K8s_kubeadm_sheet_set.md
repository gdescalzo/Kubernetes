# Kubeadm clusters initialization and configuration

> **kubeadm init**: This command initializes a Kubernetes control-plane node, bootstrapping the cluster scheduler, which is the minimum viable Kubernetes cluster. It performs pre-flight checks, installs necessary components like the kube-controller-manager, Kubernetes API server, etcd, and kubelet (all the master node components), and generates essential certificates and configuration files and sets up the private network for the pods. Make sure you have firewall exceptions and changes in place on your Kubernetes hosts for traffic if needed if you are using iptables, network policy, or something else if your hosts are in a different location separated with network devices. There are a number of ports needed.

```
kubeadm init --pod-network-cidr=10.244.0.0/16 --control-plane-endpoint=10.1.149.123
```

# Adding worker nodes

> To add a worker node to your cluster, you’ll first need to generate a token on your control plane node using the kubeadm token command:

```
kubeadm token create --print-join-command
```

# Upgrading your Kubernetes cluster

> Performing lifecycle maintenance on your Kubernetes cluster is an important part of Kubernetes cluster management. One feature of Kubeadm is it includes a command to check for updates to your Kubernetes cluster and perform those upgrades.

> To check your cluster for upgrades, use the command:

```
kubeadm upgrade plan
```

## Applying the upgrade

> After you have checked for upgrades, you can apply the upgrades using the kubeadm command:

```
kubeadm upgrade apply vX.XX.x
```

# Resetting the Cluster

> The process to reset the cluster is a simple command with kubeadm.

```
kubeadm reset
```

> Use this command to remove all Kubernetes components installed by kubeadm. It’s a useful step for starting over or cleaning up if you have an issue or errors.
