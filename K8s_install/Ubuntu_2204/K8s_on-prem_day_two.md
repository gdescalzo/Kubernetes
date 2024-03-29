
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

### Creating the Service Account and ClusterRoleBinding

```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF
```

### Get a Bearer Token

> Now we need to find token we can use to log in. Execute following command:

> ```
> kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
> ```

>It should print the data with line like:
>
> ```
> token: <YOUR TOKEN HERE>
> ```
> Now save it. You need to use it whe login the dashboard.

### Create the ingress controller

> ```
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: kubernetes-dashboard
  name: kubernetes-dashboard-ingress
  annotations:
    kubernetes.io/spec.ingressClassName.class: "nginx"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTP"
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    nginx.ingress.kubernetes.io/auth-realm: 'Authentication Required'
spec:
  rules:

- host: lolpro11.me
    http:
      paths:
  - path: /
        pathType: Prefix
        backend:
          service:
            name: kubernetes-dashboard
            port:
              number: 80
EOF
> ```
