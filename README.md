
# 🚀 Jenkins CI/CD Pipeline on AWS EC2 - Full Project Setup with Slave Nodes and GitHub Webhook

This guide demonstrates how to set up a Jenkins-based CI/CD pipeline on **AWS EC2 (Amazon Linux)** with:
- Master-slave Jenkins architecture
- GitHub integration using webhooks
- Deployment of a website through pipeline on Slave 1
- Instant code update via GitHub reflected live

---

## 🧩 Architectural Overview

![Architecture Diagram](1.png)

**Flow:**
1. Developer pushes code to **GitHub**
2. GitHub webhook triggers Jenkins Master on EC2
3. Jenkins Master assigns build to **Slave 1**
4. Slave 1 builds and hosts the website (port 85 enabled)
5. Slave 2 acts as an additional node for scalability or future pipeline stages

---

## 🔐 EC2 Instance Configuration

### Jenkins Master EC2:
- OS: Amazon Linux 2023
- Instance Type: t2.medium
- Security Group Rules:
  - TCP 22: SSH
  - TCP 80: HTTP
  - TCP 8080: Jenkins UI
  - TCP 40933: Jenkins agents

![Security Group](4.png)

---

## ⚙️ Install Jenkins using User Data

While launching the EC2 Master, add this User Data:

![User Data](11.jpeg)

```bash
#!/bin/bash
sudo dnf update
sudo dnf install java-17-amazon-corretto -y
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
sudo dnf install jenkins -y
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

---

## 🔑 Unlock Jenkins First Login

Visit `http://<Jenkins-Master-Public-IP>:8080`

![Unlock Jenkins](12.jpeg)

Use the password from EC2 instance:

![Initial Password from EC2](13.png)

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

---

## 🧱 Install Jenkins Plugins

Install suggested plugins during setup:

![Plugins Install](14.png)

---

## 🔌 Configure GitHub Webhook

In your GitHub repository:
- Go to Settings → Webhooks → Add webhook
- Payload URL: `http://<Jenkins-Master-IP>:8080/github-webhook/`
- Content type: `application/json`
- Enable push events

![GitHub Webhook](15.png)

---

## 🔀 Configure Jenkins Master-Slave Connection

### TCP Port for Inbound Agents

![Agent TCP Port](16.jpeg)

- Go to **Manage Jenkins > Configure Global Security**
- Enable and set **TCP port for agents** to a fixed value, e.g., `40933`

### Add Slave 1 Node

- Go to **Manage Jenkins > Nodes > New Node**
- Set name: `Slave1`
- Remote directory: `/home/ec2-user`
- Launch method: Connect from agent to controller

![Slave Node Setup](17.jpeg)

---

## 📦 Create Jenkins Pipeline

1. Create a new pipeline job
2. Use GitHub as SCM source
3. Sample pipeline:

```groovy
pipeline {
  agent { label 'Slave1' }
  stages {
    stage('Build') {
      steps {
        echo "The Jenkins agent is connected"
      }
    }
  }
}
```

4. Run the pipeline. Output confirms Slave1 is active:

![Jenkins Success](18.jpeg)

---

## 🌐 Website Hosting via Slave1 (Port 85)

Update the Security Group of **Slave1** to allow TCP port **85**.

- Create pipeline stage to run a local HTTP server on port 85

```groovy
stage('Serve Site') {
  steps {
    sh 'python3 -m http.server 85 &'
  }
}
```

Visit: `http://<Slave1-Public-IP>:85`

---

## 🧪 GitHub Code Update Reflects Instantly

1. Push an updated `index.html` or app file to GitHub
2. Webhook triggers Jenkins Master → triggers pipeline on Slave1
3. Output seen in browser instantly (served via updated port 85 content)

---

## 🌟 Final Highlights

- GitHub → Jenkins webhook trigger tested
- Jenkins Master on EC2 Amazon Linux with Java 17
- Master triggers Slave1 pipeline via label
- Slave1 hosts static site via Python server
- Slave2 ready as secondary agent

