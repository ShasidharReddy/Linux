# Kubernetes Day-to-Day

This guide covers daily kubectl workflows, pod troubleshooting, scaling, and node operations.

## 12.1 Common `kubectl` commands

### Context and namespace

```bash
kubectl config get-contexts
kubectl config current-context
kubectl config set-context --current --namespace=prod
```

### Get workloads

```bash
kubectl get nodes -o wide
kubectl get ns
kubectl get pods -A
kubectl get deploy -A
kubectl get svc -A
kubectl get ingress -A
```

### Describe resources

```bash
kubectl describe pod mypod -n prod
kubectl describe deploy myapp -n prod
```

## 12.2 Pod troubleshooting

### Logs

```bash
kubectl logs pod/mypod -n prod
kubectl logs pod/mypod -n prod --previous
kubectl logs deploy/myapp -n prod --tail=100
```

### Exec into pod

```bash
kubectl exec -it pod/mypod -n prod -- /bin/sh
```

### Events

```bash
kubectl get events -n prod --sort-by=.lastTimestamp
```

### Watch pod changes

```bash
kubectl get pods -n prod -w
```

## 12.3 Deployment rollouts

### Check rollout status

```bash
kubectl rollout status deploy/myapp -n prod
```

### Restart deployment

```bash
kubectl rollout restart deploy/myapp -n prod
```

### Roll back deployment

```bash
kubectl rollout undo deploy/myapp -n prod
```

### Roll back to specific revision

```bash
kubectl rollout history deploy/myapp -n prod
kubectl rollout undo deploy/myapp --to-revision=3 -n prod
```

## 12.4 Scaling applications

```bash
kubectl scale deploy/myapp --replicas=5 -n prod
```

```bash
kubectl autoscale deploy/myapp --min=2 --max=10 --cpu-percent=70 -n prod
```

## 12.5 Log aggregation from pods

### All pods by label

```bash
kubectl logs -n prod -l app=myapp --tail=100 --prefix=true
```

### Follow all matching pods one at a time

```bash
for p in $(kubectl get pods -n prod -l app=myapp -o name); do
  echo "==== $p ===="
  kubectl logs -n prod "$p" --tail=50
done
```

## 12.6 Debugging scheduling issues

```bash
kubectl describe pod mypod -n prod | egrep -A5 'Events:|Warning|FailedScheduling'
```

```bash
kubectl get nodes
kubectl describe node worker01
```

## 12.7 Resource usage

```bash
kubectl top nodes
kubectl top pods -A --containers
```

## 12.8 Common maintenance actions

### Cordon and drain node

```bash
kubectl cordon worker01
kubectl drain worker01 --ignore-daemonsets --delete-emptydir-data
```

### Uncordon node

```bash
kubectl uncordon worker01
```

## 12.9 Kubernetes troubleshooting checklist

- Check namespace.
- Check rollout status.
- Check events.
- Check logs current and previous.
- Check readiness and liveness probes.
- Check secrets and configmaps.
- Check image pull errors.
- Check node pressure.
- Check network policy.

## 12.10 Kubernetes one-liners

```bash
kubectl get pods -A -o wide | egrep 'CrashLoopBackOff|Error|Pending|ImagePullBackOff'
```

```bash
kubectl get deploy -A -o custom-columns='NS:.metadata.namespace,NAME:.metadata.name,READY:.status.readyReplicas,DESIRED:.spec.replicas,UPDATED:.status.updatedReplicas,AVAILABLE:.status.availableReplicas'
```

---
