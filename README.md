# DEZSYS Jenkins HelloSpencer

Jenkins Übung mit Docker
## Installation
- Jenkins Setup Wizard (Admin PW: `59a7f61de2524d6a9a473a1b59b9b216`)
- Plugins: Pipeline, Docker Pipeline, Docker, CloudBees Docker Build and Publish, Github Integration
- GitHub CLI im Docker Container installieren
- Pipeline erstellen
	- GITScm Polling
	- Repository anmelden
- Ngrok hosting
	- `ngrok http 8080`
- Webhook Erstellen
	- URL: `https://uninnocuous-judy-mellifluous.ngrok-free.dev/github-webhook/`
	- Type: `application/json`
## Jenkins starten
Compose File:
```yaml
services:
  jenkins:
    image: jenkins/jenkins:latest
    container_name: jenkins
    user: root
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - jenkins_home:/var/jenkins_home
```

Starten:
```bash
sudo docker compose up -d
```

## Testen

Auf GitHub pushen:
```bash
git add .
git commit -m "message"
git push
```

Jenkins beobachten:
![[jenkins-builds.png]]

Website abrufen:
```bash
curl http://localhost:5556/api/hello
```