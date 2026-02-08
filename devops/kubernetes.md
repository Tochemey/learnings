# Kubernetes Useful Commands

Quick reference for `kubectl` and cluster operations. See [DevOps README](./README.md) for more resources.

**Table of contents**

1. [Cluster & context](#cluster--context)
2. [Getting resources](#getting-resources)
3. [Pods (logs, exec, describe)](#pods-logs-exec-describe)
4. [Port forwarding](#port-forwarding)
5. [Creating & applying resources](#creating--applying-resources)
6. [Deleting & force-removing](#deleting--force-removing)
7. [Rollouts & scaling](#rollouts--scaling)
8. [Debugging & inspection](#debugging--inspection)
9. [Validation & dry-run](#validation--dry-run)
10. [Node operations](#node-operations)
11. [ConfigMaps & Secrets](#configmaps--secrets)

---

## Cluster & context

Cluster info:

```bash
kubectl cluster-info
```

Current context and namespace:

```bash
kubectl config current-context
kubectl config get-contexts
```

Use a namespace for subsequent commands (default in current context):

```bash
kubectl config set-context --current --namespace=my-namespace
```

Short `-n` is used in examples below; replace with your namespace.

---

## Getting resources

Get pods, deployments, services in a namespace:

```bash
kubectl get pods -n namespace
kubectl get deployments -n namespace
kubectl get services -n namespace
kubectl get svc -n namespace
```

Get all common resources in one go:

```bash
kubectl get all -n namespace
```

Wide output (more columns, e.g. node, IP):

```bash
kubectl get pods -n namespace -o wide
```

Get resource as YAML or JSON:

```bash
kubectl get pod pod_name -n namespace -o yaml
kubectl get deployment deploy_name -n namespace -o yaml
kubectl get pod pod_name -n namespace -o jsonpath='{.status.podIP}'
```

Watch resources (live updates):

```bash
kubectl get pods -n namespace -w
```

List with label selector:

```bash
kubectl get pods -n namespace -l app=myapp
kubectl get pods -n namespace --selector="env=prod"
```

---

## Pods (logs, exec, describe)

Stream logs from a pod:

```bash
kubectl logs pod_name -n namespace
kubectl logs -f pod_name -n namespace              # follow
kubectl logs --tail=100 pod_name -n namespace      # last 100 lines
kubectl logs -c container_name pod_name -n namespace  # specific container
```

Logs from previous (crashed) container instance:

```bash
kubectl logs pod_name -n namespace --previous
```

Execute a command in a running pod:

```bash
kubectl exec -n namespace pod_name -- ls /
kubectl exec -it -n namespace pod_name -- /bin/sh   # interactive (Alpine)
kubectl exec -it -n namespace pod_name -- /bin/bash # interactive (Debian)
kubectl exec -it -n namespace pod_name -c container_name -- /bin/sh  # multi-container
```

Describe pod (events, state, reasons):

```bash
kubectl describe pod pod_name -n namespace
```

---

## Port forwarding

Forward local port to a pod:

```bash
kubectl port-forward -n namespace pod/pod_name 8080:8080
```

Forward to a deployment (picks a pod):

```bash
kubectl port-forward -n namespace deployment/deployment_name 8080:8080
```

Forward to a service:

```bash
kubectl port-forward -n namespace svc/svc_name 8080:8080
```

Listen on a specific host:

```bash
kubectl port-forward -n namespace pod/pod_name 127.0.0.1:8080:8080
```

---

## Creating & applying resources

Create/update from YAML:

```bash
kubectl apply -f manifest.yaml
kubectl apply -f ./dir/
kubectl apply -f https://example.com/manifest.yaml
```

Create resource (fails if already exists):

```bash
kubectl create -f manifest.yaml
```

Run a one-off pod (e.g. debug):

```bash
kubectl run -it --rm debug --image=busybox --restart=Never -n namespace -- sh
kubectl run -it --rm curl --image=curlimages/curl --restart=Never -n namespace -- curl http://svc:8080
```

---

## Deleting & force-removing

Delete a resource:

```bash
kubectl delete pod pod_name -n namespace
kubectl delete -f manifest.yaml -n namespace
```

Delete pod immediately (no graceful termination):

```bash
kubectl delete pod pod_name -n namespace --grace-period=0 --force
```

Delete all pods matching a label:

```bash
kubectl delete pods -n namespace -l app=myapp
```

Delete evicted / failed pods:

```bash
kubectl get pods -n namespace | grep Evicted | awk '{print $1}' | xargs kubectl delete pod -n namespace
```

---

## Rollouts & scaling

Scale a deployment:

```bash
kubectl scale deployment deployment_name -n namespace --replicas=5
```

Rollout status and history:

```bash
kubectl rollout status deployment/deployment_name -n namespace
kubectl rollout history deployment/deployment_name -n namespace
```

Rollback to previous revision:

```bash
kubectl rollout undo deployment/deployment_name -n namespace
kubectl rollout undo deployment/deployment_name -n namespace --to-revision=2
```

Restart deployment (recreate pods):

```bash
kubectl rollout restart deployment/deployment_name -n namespace
```

Pause / resume rollout:

```bash
kubectl rollout pause deployment/deployment_name -n namespace
kubectl rollout resume deployment/deployment_name -n namespace
```

---

## Debugging & inspection

Events in namespace (often why a pod didn’t start):

```bash
kubectl get events -n namespace
kubectl get events -n namespace --sort-by='.lastTimestamp'
```

Resource usage (if metrics-server is installed):

```bash
kubectl top pods -n namespace
kubectl top pods -n namespace --sort-by=memory
kubectl top nodes
```

Patch a resource without editing full YAML:

```bash
kubectl patch deployment deployment_name -n namespace -p '{"spec":{"replicas":3}}'
```

Edit resource in place (opens default editor):

```bash
kubectl edit pod pod_name -n namespace
```

Add/remove labels:

```bash
kubectl label pod pod_name -n namespace env=prod
kubectl label pod pod_name -n namespace env-   # remove label "env"
```

Add annotations:

```bash
kubectl annotate pod pod_name -n namespace description="my note"
```

---

## Validation & dry-run

Validate and show what would be applied (no changes):

```bash
kubectl apply -f manifest.yaml --dry-run=client
kubectl apply -f manifest.yaml --dry-run=server   # server-side validation
```

Generate YAML without creating (useful for templates):

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml
```

---

## Node operations

List nodes and their status:

```bash
kubectl get nodes
kubectl get nodes -o wide
kubectl describe node node_name
```

Mark node as unschedulable (cordon):

```bash
kubectl cordon node_name
```

Drain node (evict pods so you can maintain the node):

```bash
kubectl drain node_name --ignore-daemonsets --delete-emptydir-data
```

Allow scheduling again:

```bash
kubectl uncordon node_name
```

---

## ConfigMaps & Secrets

Create ConfigMap from literal or file:

```bash
kubectl create configmap my-cm -n namespace --from-literal=key=value
kubectl create configmap my-cm -n namespace --from-file=app.properties
```

Create generic secret (base64-encoded):

```bash
kubectl create secret generic my-secret -n namespace --from-literal=password=secret
kubectl create secret generic my-secret -n namespace --from-file=./tls.crt
```

List and inspect:

```bash
kubectl get configmaps -n namespace
kubectl get secrets -n namespace
kubectl get secret my-secret -n namespace -o jsonpath='{.data.password}' | base64 -d
```

---

## See also

- [Docker](./docker.md) — image and container commands
- [Tips](./tips.md) — production survival guide (probes, rollbacks, observability)
- [Network Policies](https://github.com/Tochemey/kubernetes-network-policy-recipes) - kubernetes network policies
