Setup and verification

Prerequisites:
- A Kubernetes cluster (minikube / kind / real cluster)
- `kubectl` configured
- The namespace `devops` created: `kubectl create ns devops`

Apply the manifests (prometheus and grafana files are already present):

```bash
kubectl apply -f monitoring/prometheus.yaml
kubectl apply -f monitoring/grafana.yaml
# optional: apply the multi-container pod (alternative single-pod approach)
kubectl apply -f monitoring/monitoring-pod.yaml
```

Verify Pod running (screenshot requirement):

```bash
kubectl get pods -n devops
kubectl describe pod monitoring-pod -n devops
```

Access the Spring Actuator metrics endpoint (screenshot requirement):
- If using the provided `spring-deployment.yaml` with NodePort 30080, from a cluster node or using `minikube service`:

```bash
# from local host when NodePort exposed on 30080
# note: the application uses context-path `/student` so actuator endpoint is `/student/actuator/prometheus`
curl http://<node-ip>:30080/student/actuator/prometheus
# or using minikube
minikube service spring-service -n devops --url
# then curl <url>/student/actuator/prometheus
```

Prometheus UI (screenshot requirement):
- If Prometheus exposed as NodePort (prometheus.yaml uses NodePort), find its node port:

```bash
kubectl get svc prometheus-service -n devops
```

- Visit `http://<node-ip>:<prometheus-nodeport>` and open `Status -> Targets` to confirm the Spring Boot target is scraped.

Grafana (screenshot requirement and dashboard import):
- Get Grafana NodePort and open UI (`admin` / `admin` or password set in manifest):

```bash
kubectl get svc grafana-service -n devops
```

- In Grafana: Add a Prometheus datasource pointing to `http://prometheus-service:9090` (if Grafana running inside cluster) or `http://<node-ip>:<prometheus-nodeport>` from your machine.
- Import the dashboard: `monitoring/grafana-dashboard.json` via Grafana UI -> Manage -> Import.

Deliverables (screenshots to take):
- `kubectl get pods -n devops` showing Prometheus/Grafana and Spring pods running.
- `curl http://<spring-url>/actuator/prometheus` showing plaintext metrics.
- Prometheus UI showing the Spring target in `Status -> Targets`.
- Grafana showing the imported dashboard with graphs.

Notes:
- `monitoring/prometheus.yaml` already contains the `ConfigMap` named `prometheus-config` with `prometheus.yml` that scrapes `spring-service:8080` on `/actuator/prometheus`.
- If your cluster DNS or service names differ, update `prometheus.yml` targets accordingly.
