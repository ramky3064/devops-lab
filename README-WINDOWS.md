# Windows notes

- `Jenkinsfile` = Windows (bat). `Jenkinsfile.linux` = original Linux (sh) version.
- Install: Python 3.12 (tick "Add to PATH" / system-wide), Git, Docker Desktop, JDK 17.
- Local test (PowerShell):
    python -m venv venv
    .\venv\Scripts\Activate.ps1      # if blocked: Set-ExecutionPolicy -Scope Process Bypass
    pip install -r requirements-dev.txt
    pytest tests --cov=app
- Jenkins service must see Python, Git and Docker on the SYSTEM PATH (restart the Jenkins service after installing).
- If `docker` gives "access denied" in a build: services.msc > Jenkins > Properties > Log On > run as your Windows user, restart.
- Sonar webhook URL (Sonar is in Docker Desktop): http://host.docker.internal:8080/sonarqube-webhook/
- Jenkins -> Sonar URL stays http://localhost:9000
