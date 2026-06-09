# DEZSYS Jenkins HelloSpencer

A Flask REST API used as a CI/CD learning exercise with Jenkins and Docker.

## Application

`GET /api/hello` returns a JSON greeting and a persistent hit counter:

```json
{ "message": "Hello Spencer", "counter": 42, "status": "success" }
```

The counter is stored in `count.txt` and increments on every request.

---

## Setup

### 1. Install Jenkins

Run Jenkins in Docker:

```bash
docker run -u root -d -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins/jenkins:latest
```

Open `http://localhost:8080` and complete the setup wizard.

### 2. Install Jenkins Plugins

Go to **Manage Jenkins → Plugins** and install:
- Docker plugin
- CloudBees Docker Build and Publish
- GitHub Integration Plugin

Restart Jenkins after installation.

### 3. Verify Docker in Jenkins

```bash
docker exec -it <jenkins_container_name> bash
docker --version
```

If `docker: command not found`, install the Docker CLI inside the container:

```bash
apt-get update
apt-get install -y apt-transport-https ca-certificates curl gnupg lsb-release
curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
apt-get update
apt-get install -y docker-ce-cli
```

---

## HelloWorld Pipeline (starting point)

Before setting up the full application pipeline, create a minimal pipeline to verify Jenkins works:

1. **New Item** → Pipeline → name it `HelloWorld`
2. Under *Pipeline*, select *Pipeline script* and paste:

```groovy
pipeline {
    agent any
    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

3. Click **Build Now** and confirm the build succeeds in the console output.

---

## Full Application Pipeline

### 4. Create the Pipeline

1. **New Item** → Pipeline → name it `HelloSpencer`
2. Under *Build Triggers*, check **GitHub hook trigger for GITScm polling**
3. Under *Pipeline*, select **Pipeline script from SCM**
   - SCM: Git
   - Repository URL: `https://github.com/ThomasMicheler/DEZSYS_JENKINS_HELLOSPENCER.git`
   - Branch: `*/main`
   - Script Path: `Jenkinsfile`
4. Save

### 5. Configure GitHub Webhook

In your GitHub repo → **Settings → Webhooks → Add webhook**:
- Payload URL: `http://<jenkins-ip>:8080/github-webhook/`
- Content type: `application/json`
- Trigger: *Just the push event*

> For local Jenkins, expose it with `ngrok http 8080` and use the ngrok URL.

### 6. Trigger the Pipeline

Push any commit to the `main` branch. Jenkins will automatically start the pipeline.

---

## Pipeline Stages

| Stage | Description |
|---|---|
| Pre-Build Cleanup | Kills any leftover Flask processes |
| Checkout | Clones the repo from GitHub |
| Build | Installs Python dependencies |
| Test | Runs unit tests (`tests/test_hello.py`) |
| Run | Starts the Flask server and smoke-tests it with curl |
| Test API | Runs live integration tests (`tests/test_api.py`) |
| Deploy | Builds the Docker image and runs it as `hellospencer` container |

---

## Running Locally

```bash
pip install -r requirements.txt
echo "0" > count.txt
python src/hello.py
curl http://localhost:5556/api/hello
```

## Running Tests

```bash
# Unit tests (no server needed)
python -m pytest tests/test_hello.py -v

# Integration tests (requires running server)
python -m pytest tests/test_api.py -v
```

## Docker

```bash
docker build -t hellospencer .
docker run -d --name hellospencer -p 5556:5556 hellospencer
curl http://localhost:5556/api/hello
```
