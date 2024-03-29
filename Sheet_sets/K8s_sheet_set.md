# Kubernetes Command Cheat Sheet

## Retrieving Resources

| Command | Description |
| - | - |
| `kubectl get nodes` | Lists all nodes in the cluster |
| `kubectl get pods` | Lists all pods in the current namespace |
| `kubectl get services` | Lists all services in the current namespace |

## Creating and Applying Configurations

| Command | Description |
| - | - |
| `kubectl apply -f file.yaml` | Applies the configuration defined in a YAML file |
| `kubectl create -f file.yaml` | Creates a resource from a YAML file. |

## Inspecting and Describing Resources

| Command | Description |
| - | - |
| `kubectl describe node <node-name>` | Displays detailed information about a node. |
| `kubectl describe pod <pod-name>` | Displays detailed information about a pod. |

## Executing Commands Inside Containers

| Command | Description |
| - | - |
| `kubectl exec -it <pod-name> -- <command>` | Executes a command inside a container in a pod. |

## Viewing Logs

| Command | Description |
| - | - |
| `kubectl logs <pod-name>` | Displays the logs from a pod. |

## Deleting Resources

| Command | Description |
| - | - |
| `kubectl delete -f file.yaml` | Deletes resources defined in a YAML file. |
| `kubectl delete pod <pod-name>` | Deletes a specific pod. |

## Port Forwarding

| Command | Description |
| - | - |
| `kubectl port-forward pod/<pod-name> <local-port>:<remote-port>` | Forwards a local port to a port on the pod. |

## Scaling Applications

| Command | Description |
| - | - |
| `kubectl scale --replicas=<number> deployment/<deployment-name>` | Changes the number of replicas of a deployment. |

## Updating Images

| Command | Description |
| - | - |
| `kubectl set image deployment/<deployment-name> <container-name>=<new-image>` | Updates the image of a container in a deployment. |
