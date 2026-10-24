# Task 2: Simple Jenkins Pipeline for CI/CD

A minimal CI/CD pipeline using **Jenkins** and **Docker** that builds, tests and deploys a small Node.js app on every commit.

## Project structure

| File | Purpose |
|------|---------|
| `app.js` | Tiny HTTP app (`/` and `/health`) |
| `test/app.test.js` | Unit test run in the Test stage |
| `Dockerfile` | Packages the app into an image |
| `Jenkinsfile` | Declarative pipeline (Checkout, Build, Test, Deploy, Verify) |

## Pipeline stages

1. **Checkout**: pulls the latest code from the repo.
2. **Build**: builds a Docker image tagged with the build number and `latest`.
3. **Test**: runs `npm test` inside the freshly built image.
4. **Deploy**: removes the old container and starts the new one on port 3000.
5. **Verify**: calls `/health` to confirm the deployment works.

## Setup

### 1. Run Jenkins (Docker, free)
```bash
docker run -d --name jenkins -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins/jenkins:lts
```
Then install the Docker CLI inside the container (or use a custom image):
```bash
docker exec -u root jenkins sh -c "apt-get update && apt-get install -y docker.io curl"
docker exec -u root jenkins chmod 666 /var/run/docker.sock
```
Open http://localhost:8080, unlock with the password from `docker logs jenkins`, and install the suggested plugins.

### 2. Create the pipeline job
- New Item, name it `cicd-demo`, choose **Pipeline**.
- Under *Pipeline*, select **Pipeline script from SCM**, SCM = Git, paste this repo's URL, branch `*/main`, Script Path = `Jenkinsfile`.

### 3. Trigger on every commit
- **Option A (webhook):** GitHub repo, Settings, Webhooks, add `http://<jenkins-url>:8080/github-webhook/` (use ngrok if Jenkins is local). Tick *GitHub hook trigger for GITScm polling* in the job.
- **Option B (polling):** already configured in the Jenkinsfile via `pollSCM('H/2 * * * *')`.

### 4. Test it
Push a change (e.g. edit the greeting in `app.js`), watch the build run in the Jenkins dashboard, then open http://localhost:3000.

## Screenshots
Add screenshots of the successful pipeline run (Stage View) and the running app here.

## Interview Q&A

**1. What is Jenkins, and how is it used in CI/CD?**
Jenkins is an open-source automation server. In CI it automatically builds and tests code on every commit; in CD it deploys the tested build to servers or containers.

**2. What is a Jenkinsfile?**
A text file, stored in the repo, that defines the pipeline as code (stages and steps), so it is versioned and reviewed like any other code.

**3. How do you create and configure Jenkins pipelines?**
Install Jenkins and plugins, create a Pipeline job, point it to the repo's Jenkinsfile (Pipeline script from SCM), add credentials, and configure triggers (webhook or poll SCM).

**4. Common stages?**
Checkout, Build, Test, Static analysis / code quality, Package (e.g. Docker image), Push to registry, Deploy, Verify/Notify.

**5. Declarative vs Scripted pipeline?**
Declarative uses a structured `pipeline { }` syntax with predefined sections (agent, stages, post), is simpler and validated up front. Scripted is Groovy-based (`node { }`), more flexible and powerful, but harder to read and maintain.
