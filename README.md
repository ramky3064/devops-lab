# DevOps Lab: Task API (Flask)

Pipeline: GitHub -> Jenkins -> pytest -> SonarQube -> Quality Gate -> Docker -> run -> Prometheus -> Grafana

## 0. Run locally first (sanity check)
```bash
python3 -m venv venv && . venv/bin/activate
pip install -r requirements-dev.txt
pytest tests --cov=app
python -m app.main          # http://localhost:5000/health , /metrics
```

## 1. Git
```bash
git init && git add . && git commit -m "initial lab project"
git remote add origin <your-repo-url> && git push -u origin main
```

## 2. Jenkins prerequisites (native install)
- `sudo apt install python3-venv curl docker.io`
- `sudo usermod -aG docker jenkins && sudo systemctl restart jenkins`
- Plugins: Pipeline, Git, JUnit, **SonarQube Scanner**, (optional) Prometheus metrics
- Manage Jenkins -> Tools -> add **SonarQube Scanner** named `SonarScanner` (auto-install)
- In SonarQube (http://localhost:9000): My Account -> Security -> generate a token
- Jenkins -> Credentials -> add "Secret text" with that token (ID `sonar-token`)
- Manage Jenkins -> System -> SonarQube servers -> Name `sonarqube`, URL `http://localhost:9000`, token credential above
- SonarQube -> Administration -> Configuration -> Webhooks -> add `http://<host-ip>:8080/sonarqube-webhook/`
  (needed so `waitForQualityGate` returns; use the host's LAN IP, not localhost, since Sonar runs in Docker)
- New Item -> Pipeline -> "Pipeline script from SCM" -> your repo -> `Jenkinsfile`

## 3. Prometheus + Grafana
- Append `prometheus/prometheus-snippet.yml` to your `prometheus.yml`, then `sudo systemctl restart prometheus`
- Check http://localhost:9090/targets shows `taskapi` UP
- Grafana: add Prometheus data source, build panels from `prometheus/grafana-queries.md`
- Generate traffic:
```bash
for i in $(seq 1 200); do curl -s -X POST localhost:5000/tasks -H 'Content-Type: application/json' -d '{"title":"t'$i'"}' >/dev/null; curl -s localhost:5000/tasks/9999 >/dev/null; done
```

## 4. Lab experiments (what to show)
1. **Green run**: all stages pass, test report and coverage in Jenkins.
2. **Broken test**: edit an assertion -> pipeline fails at Unit Tests.
3. **Quality gate failure**: add an unused variable, a hard-coded password string, or delete tests so coverage drops -> Sonar gate fails.
4. **Monitoring**: show request-rate and 404 spikes in Grafana during the curl loop.

## 5. Optional: Ansible
```bash
cd ansible
ansible-galaxy collection install community.docker
ansible-playbook -i inventory.ini deploy.yml --limit local
```

## 6. Optional: GCP (after everything above works)
```bash
cd terraform
terraform init
terraform apply -var project_id=<YOUR_PROJECT_ID>
```
Then push the image (Artifact Registry) or build on the VM, add the VM IP to `ansible/inventory.ini`, and run the playbook against `gcp`.
Remember `terraform destroy` when done to avoid charges.
