# Jenkins Pipeline Job
Step-by-step process for creating and running a Jenkins Pipeline Job that automates building and running a Docker container, just like in your Docker foundations project.
---

#  Jenkins Pipeline Job Project

## 1. What is a Jenkins Pipeline Job?

A **Jenkins Pipeline Job** is a job that defines the **entire CI/CD workflow as code**.
Instead of clicking buttons in the Jenkins UI for each step, you can write a script (called a **Jenkinsfile**) that:

* Pulls code from GitHub
* Builds a Docker image from a Dockerfile
* Runs the container automatically

This makes builds **repeatable, automated, and version-controlled**.

---

## 2. Creating the Pipeline Job

1. Go to your **Jenkins Dashboard**.
2. On the left menu, click **New Item**.
3. Enter a name → `My pipeline job`.
4. Select **Pipeline** and click **OK**.
5. You’ll now be inside the **job configuration page**.

---

## 3. Configure Build Trigger

We want Jenkins to build automatically when changes are pushed to GitHub.

1. Inside the job config, scroll down to **Build Triggers**.
2. Select **GitHub hook trigger for GITScm polling**.
3. Since you already created a **GitHub webhook** in an earlier project, no need to create a new one.

✔ Now Jenkins will auto-trigger when you push code to GitHub.

---

## 4. Writing the Pipeline Script

Paste the following into the **Pipeline Script** section of the job configuration (or save it in a `Jenkinsfile` inside your GitHub repo):

```groovy
pipeline {
    agent any

    stages {
        stage('Connect To Github') {
            steps {
                // Checkout the repository from GitHub
                checkout scmGit(
                    branches: [[name: '*/main']],
                    extensions: [],
                    userRemoteConfigs: [[url: 'https://github.com/RidwanAz/jenkins-scm.git']]
                )
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image from Dockerfile in repo
                    sh 'docker build -t my-docker-app .'
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    // Run Docker container and map port 8081 -> 80
                    sh 'docker run -itd -p 8081:80 my-docker-app'
                }
            }
        }
    }
}
```

---

## 5. Explanation of Script

###  Agent

```groovy
agent any
```

Runs the pipeline on **any available agent (node)**.

###  Stage 1: Connect to GitHub

```groovy
checkout scmGit(branches: [[name: '*/main']], userRemoteConfigs: [[url: 'https://github.com/RidwanAz/jenkins-scm.git']])
```

* Clones the repository.
* Uses the `main` branch.

###  Stage 2: Build Docker Image

```groovy
sh 'docker build -t my-docker-app .'
```

* Builds a Docker image with tag `my-docker-app`.
* Uses the `Dockerfile` from the repo root.

###  Stage 3: Run Docker Container

```groovy
sh 'docker run -itd -p 8081:80 my-docker-app'
```

* Runs the container in detached mode.
* Maps container port `80` to host port `8081`.
* Your app will be accessible at:
   `http://<jenkins-server-ip>:8081`

---

## 6. Running the Job

1. Save the pipeline configuration.
2. Click **Build Now**.
3. Check the **console output** to confirm each stage runs:

   * GitHub repo cloned
   * Docker image built
   * Container started
4. Visit `http://<jenkins-server-ip>:8081` in your browser.

---

## 7. Optional Improvements

* **Clean up old containers** before running new ones:

```groovy
sh 'docker rm -f $(docker ps -aq) || true'
```

* **Push image to Docker Hub** (for deployment across servers).
* **Use credentials** in Jenkins for private repos or registries.
* **Add Test Stage** to run unit tests before building Docker image.

---
