# kubectl: Interview Essentials

Use commands to test a hypothesis. Explain what you expect to see, what the output establishes, and what you would check next.

Examples use namespace `interview`, pod `scanner-abc`, container `worker`, node `worker-1`, and Deployment `scanner`. Replace them with the supplied names. After confirming the intended context, use `--context <context>` explicitly if working across clusters.

## 1. Orient

| Example | When to use it |
|---|---|
| `kubectl config current-context` | Confirm you are targeting the intended cluster before investigating or changing anything. |
| `kubectl get pods -n interview -o wide` | Find affected pods, readiness, restarts, age, and node placement. |

## 2. Find the failure

| Example | When to use it |
|---|---|
| `kubectl describe pod scanner-abc -n interview` | Read conditions, container exit/waiting reasons, and events to identify the blocking symptom. |
| `kubectl events -n interview --for pod/scanner-abc` | Focus on recent scheduling, image pull, probe, or volume events. |
| `kubectl get pod scanner-abc -n interview -o yaml` | Inspect the admitted configuration, including images, requests, probes, placement rules, and mounts. |

**Interview point:** Pending includes scheduling and container setup; check node assignment and conditions before choosing a branch.

## 3. Investigate the application

| Example | When to use it |
|---|---|
| `kubectl logs scanner-abc -n interview -c worker --since=10m --tail=100 --timestamps` | Read application errors around the incident and name the relevant container explicitly. |
| `kubectl logs scanner-abc -n interview -c worker --previous --tail=100` | Inspect the previous instance of a restarted container. |
| `kubectl exec -n interview scanner-abc -c worker -- curl -fsS --max-time 5 http://localhost:8080/healthz` | Test the application inside the container, if that image contains curl and the endpoint applies. |

**Interview point:** A missing shell or diagnostic tool does not mean the application is broken. An exec command can have side effects depending on what you run.

## 4. Check dependencies and capacity

| Example | When to use it |
|---|---|
| `kubectl get svc,endpointslices -n interview` | Check whether a Service has the expected backends; compare its selector, ports, and endpoint readiness. |
| `kubectl get pvc -n interview` | Identify claims needing investigation, then describe the affected claim for binding or provisioning events. |
| `kubectl describe node worker-1` | Inspect node conditions, taints, allocatable resources, and existing resource requests when placement or node health is suspect. |
| `kubectl top pods -n interview --containers` | Check recent CPU and memory usage when the resource metrics API is available. |

**Interview point:** Scheduling uses requests, not live usage. A Running pod or a populated EndpointSlice alone does not prove customer traffic succeeds. Kubernetes permissions and AWS IAM permissions are separate investigations.

## 5. Recover and verify

| Example | When to use it |
|---|---|
| `kubectl rollout status deployment/scanner -n interview --timeout=120s` | Check whether the Deployment completes its rollout. |
| `kubectl rollout history deployment/scanner -n interview` | Inspect available revisions before selecting a recovery action. |
| `kubectl rollout undo deployment/scanner -n interview --to-revision=2` | Restore a revision you have established is known-good, when rollback is authorized and appropriate. |

**Interview point:** Undo changes the workload. Check compatibility and the deployment source first; rollback does not automatically restore external dependencies or separately changed configuration. Afterward, verify the customer operation or expected scan output as well as rollout health.

## Three practice drills

1. **Pod will not start:** Inspect conditions and events → distinguish scheduling from setup → identify the blocking condition → propose a targeted fix.
2. **Pod keeps restarting:** Inspect termination reason and probe events → read previous logs → test the leading hypothesis.
3. **Pod runs, but the application fails:** Check readiness and application logs → inspect Service backends or the failing dependency → reproduce the operation from the relevant context.

For each drill, finish with: **impact → evidence → cause or hypothesis → mitigation → verification**.

## Sources

- [Kubernetes kubectl quick reference](https://kubernetes.io/docs/reference/kubectl/quick-reference/)
- [Pod lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)
- [Container logs](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_logs/)
- [Service troubleshooting](https://kubernetes.io/docs/tasks/debug/debug-application/debug-service/)
- [Resource metrics](https://kubernetes.io/docs/tasks/debug/debug-cluster/resource-metrics-pipeline/)
- [Deployment rollbacks](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-back-a-deployment)
