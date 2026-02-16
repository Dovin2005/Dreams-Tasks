

✅ Install Docker

✅ Run Jenkins in Docker (with Docker access)

✅ Run Prometheus using Jenkins pipeline

I’ll give you a complete production-style setup.

🧱 STEP 1 — Install Docker (On Host: WSL / EC2)

If Docker is not installed:

sudo apt update
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker

Test:

docker --version
docker ps

If this works → Docker engine is ready.

🧱 STEP 2 — Get Docker Group ID (Important)

Run:

getent group docker

Example output:

docker:x:998:

Note the number (example: 998).

We’ll use it in Dockerfile.

🧱 STEP 3 — Create Jenkins Custom Image (With Docker CLI)

Create folder:

mkdir jenkins-docker
cd jenkins-docker
nano Dockerfile

Paste this (replace 998 with your docker GID):

FROM jenkins/jenkins:lts

USER root

RUN apt-get update && \
    apt-get install -y docker.io && \
    apt-get clean

# Match host docker group ID
RUN groupadd -g 998 docker || true && \
    usermod -aG docker jenkins

USER jenkins

Save.

🧱 STEP 4 — Build Jenkins Image
docker build -t jenkins-with-docker .

Wait until complete.

🧱 STEP 5 — Run Jenkins Container
docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins-with-docker

Now open:

http://localhost:8080

If on EC2:

http://EC2_PUBLIC_IP:8080

Make sure security group allows 8080.

🧱 STEP 6 — Install Required Jenkins Plugins

Go to:

Manage Jenkins → Plugins

Install:

Docker Pipeline

Git

SSH Agent

Restart Jenkins.

🧱 STEP 7 — Verify Docker Works Inside Jenkins

Inside container:

docker exec -it jenkins bash
docker ps
groups

If it works → setup is correct.

🚀 STEP 8 — Prometheus Jenkins Pipeline

Create a new Pipeline job and paste:

pipeline {
    agent any

    stages {

        stage('Pull Prometheus Image') {
            steps {
                sh 'docker pull prom/prometheus'
            }
        }

        stage('Run Prometheus Container') {
            steps {
                sh '''
                docker stop prometheus || true
                docker rm prometheus || true
                docker run -d -p 9090:9090 --name prometheus prom/prometheus
                '''
            }
        }
    }
}

Click Build Now.

📊 STEP 9 — Access Prometheus

Open:

http://localhost:9090



http://EC2_PUBLIC_IP:9090

Make sure port 9090 is open in security group.
