
## 🖥️ 9 - Configure Slave1 EC2 Instance and Agent Connection

- Launch a new EC2 Instance named **Jenkins Slave1**
- Use Amazon Linux 2 and add the following commands to **User Data** for setup:

```bash
#!/bin/bash
sudo yum update -y
sudo yum install java-1.8.0-openjdk -y
sudo yum install docker -y
sudo systemctl start docker
sudo usermod -aG docker ec2-user
```

📸 *Image 9.0: Slave1 EC2 Setup — `3.jpeg`*

---

## 🔌 10 - Add Node Entry for Slave1 in Jenkins

- In Jenkins → Manage Jenkins → Nodes → New Node
- Enter: `slave1`, choose Permanent Agent
- Add SSH credentials, use same Public IP and port

📸 *Image 10.0: Node Entry Configuration — `22.jpeg`*

---

## 🔧 11 - Create GitHub Integration Pipeline

- Create new Jenkins Pipeline job: `GitHub-CI-Test`
- Add GitHub Repository URL under Source Code Management (SCM)
- Use `main` branch

📸 *Image 11.0: GitHub Repo Config in Jenkins — `23.jpeg`*

- Add pipeline script with Docker build steps

📸 *Image 11.1: Pipeline Dockerfile & Configuration — `24.jpeg`*

- Save and build → Green success tick

📸 *Image 11.2: Successful CI Build in Jenkins — `26.jpeg`*

---

## 🌐 12 - Add Webhook in GitHub Repository Settings

- Go to GitHub → Repo Settings → Webhooks → Add Webhook
- Payload URL: `http://<jenkins_public_ip>:8080/github-webhook/`
- Content type: `application/json`

📸 *Image 12.0: GitHub Webhook Config — `4.png`*

📸 *Image 12.1: GitHub Webhook Delivery Example — `5.jpg`*

---

## ⚙️ 13 - Final DevOps Pipeline Execution (Production Build)

- Create pipeline: `DevOps-Production`
- Add same GitHub repository & credentials

📸 *Image 13.0: DevOps Pipeline Setup — `22.jpeg`*

- Add Jenkinsfile script with Docker image build:

```bash
docker build -t gagan .
docker run -d -p 85:80 gagan
```

📸 *Image 13.1: Jenkinsfile Command Execution — `24.jpeg`*

- Successful pipeline run confirmed

📸 *Image 13.2: Green Success Status — `26.jpeg`*

- Jenkins Dashboard now shows all pipelines:

📸 *Image 13.3: Jenkins Dashboard Overview — `29.jpeg`*

---

## 🌍 14 - Deploy and Access Static Website

- Go to AWS EC2 → Jenkins Slave1 → Copy Public IP
- Edit Security Group → Inbound Rules → Add Custom TCP: Port 85

📸 *Image 14.0: Public IP Display (Slave1) — `25.jpeg`*

- Open browser → Visit: `http://<slave1_ip>:85`
- Static Website should now be live

---

## ✏️ 15 - Real-time Code Update and Reflect

- Go to GitHub repo → Edit HTML (e.g. Change title “shamrock” to “convent”)
- Commit changes → Jenkins webhook triggers new build
- Website instantly reflects updated content

📸 *Image 15.0: Updated Website Header — `33.jpeg`*

📸 *Image 15.1: Final Website with Updated Title — `34.jpeg`*

---

## 🌟 Final Highlights

- ✅ End-to-end CI/CD pipeline built using **Jenkins**, **GitHub**, **Docker**, and **EC2**
- 🔄 Real-time updates with **GitHub Webhooks** triggering Jenkins builds
- 🚀 Static site deployment on **EC2 Slave1** with port 85 exposed
- 🖥️ Confirmed content updates pushed to GitHub reflected instantly live

---

## 🔮 Future Steps

- ☁️ Integrate **S3 & CloudFront** for production-level static hosting
- 🔐 Configure **SSL with Let's Encrypt** and reverse proxy using **NGINX**
- 📊 Add monitoring with **Prometheus & Grafana**
- 🛡️ Harden Jenkins setup with **IAM roles** and **backup via S3/EBS**
