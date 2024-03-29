
# Nodes Configuration

## Join Workers to Master nodes

> The **worker** nodes should needs to be added to the **master** node, the way to do it is trough a command that permit share with the worker node a token, an IP of the master node, like this, on the master node:

```
sudo kubeadm join [Your IP]:6443 --token ykjrut.6ep7mjsgk524h0qq --discovery-token-ca-cert-hash sha256:8c4164fdd263babedeedca72883ee1f6f35aeed2f9584791c49c7fe234956d30
```

> Normally when you finish the on-prem installation that line with the required Token to add the workers to the master node, is shared with you at the end of the installation. But if for some reason you loose that line, the way that you can obtain the Token is like this:

```
$ kubeadm token create --print-join-command
```

## Tagging the nodes

> The nodes tagging is useful for identify the role that you are give for that particular node in the cluster, you can do that with the following command, on the master node

> ```
> kubectl label node {here your worker host name} node-role.kubernetes.io/worker=worker
> ```

> | ![k8s-tag.png](.\img\k8s-tag.png) |
> | :-: |
> | _Tagged nodes example_ |

# Installing tools 

## Install the metric service

> Execute on the Master node:

> ```
> kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
>
> kubectl edit deployment metrics-server -n kube-system
>
> Add in arg: - --kubelet-insecure-tls
> ```

> | ![k8s-top.png](.\img\k8s-top.png) |
> | :-: |
> | _Top nodes example_ |

## Install Kubernetes Dashboard

> Execute on the Master node:

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```