
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

> | ![k8s-tag.png](https://github.com/gdescalzo/Kubernetes/blob/main/K8s_install/Ubuntu_2204/img/k8s-tag.png) |
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

> | ![k8s-ingress-arch.png](.\img\k8s-ingress-arch.png) |
> | :-: |
> | _Top nodes example_ |


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

### Getting a Bearer Token for ServiceAccount


### Get a Bearer Token

```
kubectl -n kubernetes-dashboard create token admin-user
```

It should print something like:

```
gdescalzo@master:~$ kubectl -n kubernetes-dashboard create token admin-user
eyJhbGciOiJSUzI1NiIsImtpZCI6ImZSRkljSlAtWmpON2NRTHF1TVhUTWdlcXhTLWJJMGJXNGJEOEt0cXFVTVEifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzExODIwNTQ1LCJpYXQiOjE3MTE4MTY5NDUsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJhZG1pbi11c2VyIiwidWlkIjoiMmFjM2I3ZWItMWQyMy00YzgzLTliYmQtZmMyMmNlNjgwODUyIn19LCJuYmYiOjE3MTE4MTY5NDUsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlcm5ldGVzLWRhc2hib2FyZDphZG1pbi11c2VyIn0.i91XF5Sbv5almOo_Yn6iWaI0qn2xm7WLR3vLUlHtrZoJ3L7sqZP5Bn6MzDaEdIUJR6ZT7zKfoKcyj4w9Dn7Moeor2-gdrKTn8pgSIWVwL-ILqhUEVp-IobMRicMNjJhgazgCDNM3fl6IphrkDDZYHPk9AUhwrT38behOrghpXVcNJTdP25Ign2LF5AF-ALUJBO05G1tfNcjIHaJu-V4eAfPKn1wk3zJy1qQN1QeCVZxKYF9vo4SEtBgegphi_yUUn659qtcRFZCe2YvYPLmkV78V2VSfFVfkw6C7OFX3XzI4lsUMym_MAoTaa5xF0bfDi9XMQoNifhx7ReauNzZ8VA

```

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

```
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
  - host: master
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
```
