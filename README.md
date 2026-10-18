
# Day 15 - Kubernetes Fundamentals

DevOps Winter Arc | Day 15/90 | Week 3: Kubernetes

## Objective

Understand how Kubernetes runs containerized applications, then deploy an
Nginx application to a local cluster using YAML manifests. Verify the
workload, scale it to three replicas, expose it with a Service, and
document the evidence.

## Architecture and key concepts

**Kubernetes** is a container orchestration platform. I declare the desired
state in YAML manifests, and Kubernetes controllers work to keep the cluster
matching that state.

| Concept | What it does | In this lab |
|---|---|---|
| Cluster | The complete Kubernetes environment | My local cluster |
| Control plane | Makes cluster-level decisions and stores state | Coordinates the desired state |
| Worker node | Runs the Pods and application workloads | Runs the Nginx Pods |
| Pod | Smallest deployable unit, holds one or more containers | One Nginx container per Pod |
| Deployment | Manages replicated Pods and rolling updates | Keeps the desired number of Nginx replicas |
| Service | Gives a stable way to reach a set of Pods | Routes traffic to the Nginx Pods |

**Docker vs Kubernetes:** Docker builds and runs containers. Kubernetes
orchestrates containerized workloads across a cluster, handling scheduling,
scaling, self-healing, and service discovery.

**Label selectors:** A Service does not point at specific Pods. It selects
Pods by label (`app: nginx`). Any Pod with a matching label becomes a
backend, so new Pods are picked up automatically and removed Pods are dropped.

## Cluster setup

I used:  kind
```bash
# kind option
kind create cluster --name winter-arc

# verify
kubectl cluster-info
kubectl get nodes
```

Result: the node status was `Ready`.

Evidence: `evidence/01-nodes.png`

## Deploy and scale Nginx

Files in this repo:

- `deployment.yaml`: Nginx Deployment, `nginx:1.27`, starts with 2 replicas
- `service.yaml`: NodePort Service on port `30080`, selector `app: nginx`

Deploy:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl get deployments
kubectl get pods -o wide
kubectl get services
```

Evidence: `evidence/02-pods.png`, `evidence/03-deployment.png`, `evidence/04-service.png`

Scale to three replicas:

```bash
kubectl scale deployment nginx-deployment --replicas=3
kubectl get deployment nginx-deployment
kubectl get pods
```


Test access:

```bash
# NodePort (if exposed through localhost)
curl http://localhost:30080

# port-forward (reliable local test)
kubectl port-forward service/nginx-service 8080:80
curl http://localhost:8080
```

## Commands and evidence

| Step | Command | Evidence |
|---|---|---|
| Check nodes | `kubectl get nodes` | `evidence/01-nodes.png` |
| List Pods | `kubectl get pods -o wide` | `evidence/02-pods.png` |
| Check Deployment | `kubectl get deployments` | `evidence/03-deployment.png` |
| Check Service | `kubectl get services` | `evidence/04-service.png` |
| Scale to 3 | `kubectl scale deployment nginx-deployment --replicas=3` | `evidence/05-scaled-3-replicas.png` |
| Access test | `curl http://localhost:8080` | `evidence/06-nginx-access.png` |
| Troubleshooting | `kubectl describe deployment nginx-deployment` | `evidence/07-describe.png` |
| Self-healing (optional) | `kubectl delete pod <name>` then `kubectl get pods -w` | `evidence/08-self-healing.png` |

## Troubleshooting/lessons learned

Troubleshooting command used: `kubectl describe deployment nginx-deployment`

Lessons learned:

- A Deployment declares the desired replica count and Kubernetes works to
  maintain it.
- A Service finds its Pods through label selectors. If the selector and Pod
  labels do not match, the Service has no endpoints.
- `kubectl describe` and `kubectl get events` are the first places to look
  when something is wrong.
- ** PORT is not working; if I forward, then that is working"


## Cleanup

```bash
kubectl delete -f service.yaml
kubectl delete -f deployment.yaml

# only if I created a kind cluster just for this lab
kind delete cluster --name winter-arc
```

## Security

No kubeconfig files, tokens, or cloud credentials are included in this
repository.
