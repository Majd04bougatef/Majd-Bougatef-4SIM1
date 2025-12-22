# Student Management — Run & Test Guide

Short, copy‑paste commands to run this project locally (in your Vagrant VM) and test the REST endpoints and monitoring stack.

## Prérequis (dans la VM)
- Vagrant VM up (`vagrant up` + `vagrant ssh`)
- Docker, kubectl, Maven, Java installed in the VM

## 1) Pull monitoring images (Prometheus, Grafana, SonarQube)
```bash
docker pull prom/prometheus:latest
docker pull grafana/grafana:latest
docker pull sonarqube:community
```

## 2) Start Prometheus (standalone, using repo monitoring config)
# If you want a quick standalone Prometheus that scrapes localhost targets:
```bash
# copy repo config (if needed)
cp /vagrant/monitoring/prometheus.yaml /home/vagrant/monitoring/prometheus.yml || true
# ensure it's a prometheus.yml file (not the K8s manifest)
sudo tee /home/vagrant/monitoring/prometheus.yml > /dev/null <<'YML'
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'spring-boot'
    metrics_path: '/student/actuator/prometheus'
    static_configs:
      - targets: ['localhost:8089']
YML

docker rm -f prometheus 2>/dev/null || true
docker run -d --name prometheus --restart unless-stopped --network host \
  -v /home/vagrant/monitoring/prometheus.yml:/etc/prometheus/prometheus.yml:ro \
  prom/prometheus:latest
```

## 3) Start Grafana
```bash
docker rm -f grafana 2>/dev/null || true
docker run -d --name grafana --restart unless-stopped --add-host=host.docker.internal:host-gateway -p 3000:3000 grafana/grafana:latest
```

Then open Grafana at http://localhost:3000 (user: `admin` / pass: `admin`).

Import dashboard JSON from `monitoring/grafana-dashboard.json` (Grafana UI → + → Import).

## 4) Start SonarQube (optional)
```bash
docker run -d --name sonarqube --restart unless-stopped -p 9000:9000 sonarqube:community
```

## 5) Deploy app & infra to Kubernetes (optional)
If you prefer k8s, apply manifests in `k8s/` (ensure `kubectl` context points to your cluster):
```bash
kubectl create namespace devops --dry-run=client -o yaml | kubectl apply -f -
kubectl apply -f k8s/ -n devops
```

Note: If you encounter PVC immutable errors for `mysql-pvc`, see `k8s/21-mysql-pvc.yaml.html` and consider backing up data then recreating the PVC/PV.

## 6) Port-forward the app for local testing (leave running)
```bash
kubectl -n devops port-forward svc/student-management-service 8089:8089
```

## 7) Test REST endpoints (examples)
```bash
# Get all students
curl -sS http://localhost:8089/student/students/getAllStudents

# Create student
curl -X POST -H "Content-Type: application/json" -d '{"firstName":"Test","lastName":"User","email":"t@example.com"}' http://localhost:8089/student/students/createStudent

# Get metrics (Actuator Prometheus)
curl -sS http://localhost:8089/student/actuator/prometheus | head -n 40
```

## 8) Generate steady traffic (to see graphs in Grafana)
Run this in a background shell to create continuous GET traffic:
```bash
while true; do curl -sS http://localhost:8089/student/students/getAllStudents >/dev/null; sleep 0.2; done &
```

Or send N POSTs to create load:
```bash
for i in $(seq 1 100); do
  curl -sS -X POST -H "Content-Type: application/json" -d "{\"firstName\":\"T$i\",\"lastName\":\"U\"}" http://localhost:8089/student/students/createStudent &
done
```

## 9) Verify Prometheus scraping
- Open http://localhost:9090 → Status → Targets → check `spring-boot` target is `UP`.
- Or query Prometheus directly:
```bash
curl -sS "http://localhost:9090/api/v1/query?query=sum(rate(http_server_requests_seconds_count[1m]))%20by%20(uri)"
```

## 10) Troubleshooting
- If Prometheus container fails to start, check `docker logs prometheus`.
- If PVC errors appear when applying k8s manifests, ensure no pods mount the PVC, or backup data and delete/recreate the PVC.
- If Grafana cannot reach Prometheus from container, run Grafana with `--network host` or add host mapping and use `http://host.docker.internal:9090` as data source URL.

---
If you want, I can also add helper scripts:
- `monitoring/generate-traffic.sh` (automatic traffic)
- `k8s/fix-mysql-pvc.sh` (assist PVC cleanup)
