# ToDo App — Kubernetes Instructions

## Docker Hub Repository

- App image: https://hub.docker.com/r/olehkovalievskyi/todoapp

## Prerequisites

- A running Kubernetes cluster (e.g., Kind, Minikube, or Docker Desktop Kubernetes)
- `kubectl` installed and configured to talk to your cluster

## Project Structure

All Kubernetes manifests are located in the `.infrastructure` folder:

- `namespace.yml` — creates the `todoapp` namespace
- `todoapp-pod.yml` — creates the ToDo application pod (with readiness and liveness probes)
- `busybox.yml` — creates a `busyboxplus:curl` pod, used for testing connectivity from inside the cluster

## 1. Apply the manifests

Apply the manifests in the following order:

```bash
kubectl apply -f .infrastructure/namespace.yml
kubectl apply -f .infrastructure/todoapp-pod.yml
kubectl apply -f .infrastructure/busybox.yml
```

Check that both pods are running:

```bash
kubectl get pods -n todoapp
```

You should see the `todoapp` and `busybox` pods with the status `Running`.

## 2. Test the application using `port-forward`

Forward a local port to the ToDo app pod:

```bash
kubectl port-forward -n todoapp pod/todoapp 8000:8000
```

Keep this command running in a terminal, then open your browser and navigate to:

http://localhost:8000

You can also check the API:

http://localhost:8000/api/

To stop port-forwarding, press `Ctrl+C` in the terminal.

## 3. Test the application using the `busyboxplus:curl` pod

Open a shell inside the `busybox` pod:

```bash
kubectl exec -it busybox -n todoapp -- sh
```

From inside the pod, use `curl` to reach the ToDo app pod directly by its pod IP
or, if a Service is configured, by its Service name:

```sh
curl http://todoapp:8000
```

If the response returns HTML content from the ToDo app (or a valid API response
from `/api/`), the application is reachable from within the cluster.

Exit the busybox shell:

```sh
exit
```

## Cleaning up

To remove all resources created by these manifests:

```bash
kubectl delete -f .infrastructure/busybox.yml
kubectl delete -f .infrastructure/todoapp-pod.yml
kubectl delete -f .infrastructure/namespace.yml
```

## Notes

- The `todoapp` pod has both readiness and liveness probes configured, which
  check the application's `/api/readiness` and `/api/liveness` endpoints.
- Database migrations run automatically at container startup.