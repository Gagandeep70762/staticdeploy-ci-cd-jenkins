# 🚀 Static Website CI/CD Pipeline Using Jenkins, GitHub, Docker & AWS EC2


This project demonstrates an end-to-end CI/CD pipeline for deploying a static website using Jenkins master-slave architecture, GitHub, and Docker on AWS EC2 instances.
![](https://github.com/Gagandeep70762/staticdeploy-ci-cd-jenkins/blob/main/images/0.jpg)
---

## 🧠 Overview


- Jenkins installed on a **t2.medium** EC2 Master (Amazon Linux 2)
- Two EC2 Slave Nodes used for offloading builds
- CI/CD triggered via GitHub Webhooks
- Docker container used for building and running static site
- Final output live on EC2 IP with exposed port

![](https://github.com/Gagandeep70762/staticdeploy-ci-cd-jenkins/blob/main/images/0.1.jpg)
![](https://github.com/Gagandeep70762/staticdeploy-ci-cd-jenkins/blob/main/images/1.png)
![](https://github.com/Gagandeep70762/staticdeploy-ci-cd-jenkins/blob/main/images/2.png)
---

## ☁️ 1 - Launch Jenkins Master EC2 Instance


- **AMI**: Amazon Linux 2
- **Instance Type**: t2.medium
- **Ports**: SSH (22), HTTP (80), TCP (8080)
![](https://github.com/Gagandeep70762/staticdeploy-ci-cd-jenkins/blob/main/images/30.png)
![](https://github.com/Gagandeep70762/staticdeploy-ci-cd-jenkins/blob/main/images/11.png)
---

## ⚙️ 2 - Install Jenkins on Master

![](https://github.com/Gagandeep70762/staticdeploy-ci-cd-jenkins/blob/main/images/12.png)

```bash
#!/bin/bash
sudo dnf update -y
sudo dnf install java-17-amazon-corretto -y
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo dnf install jenkins -y
sudo systemctl enable jenkins
sudo systemctl start jenkins
```
![](https://github.com/Gagandeep70762/staticdeploy-ci-cd-jenkins/blob/main/images/31.png)
---

## 🔐 3 - Unlock Jenkins Interface

![](https://github.com/Gagandeep70762/staticdeploy-ci-cd-jenkins/blob/main/images/13.png)

To access Jenkins setup wizard, run:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
![](https://github.com/Gagandeep70762/staticdeploy-ci-cd-jenkins/blob/main/images/15.png)
![](https://github.com/Gagandeep70762/staticdeploy-ci-cd-jenkins/blob/main/images/14.png)

Paste the copied password into the browser screen at `http://<public-ip>:8080`

---

## 🔒 4 - Open Port 8080 and Agent Port (40933) in Security Group


Additionally, to allow slave nodes to communicate with the Jenkins master over a fixed TCP port (e.g., `40933`), you must also add this port to your EC2 Security Group:

- Edit Inbound Rules → Add Custom TCP Rule with port `40933`, source `Anywhere`

This is required if you choose a fixed TCP port for inbound agents in Jenkins (as shown in the Jenkins Security settings screen).
![](https://github.com/Gagandeep70762/staticdeploy-ci-cd-jenkins/blob/main/images/16.png)
---

## 🚀 5 - Create Jenkins Slave Nodes

Go to Jenkins Dashboard → **Manage Jenkins → Nodes → New Node**:

- Name your node as `Slave1`
- Choose **Permanent Agent**
- Enter **Remote root directory** and assign **Label**
- Use Public IP and SSH credentials to connect

This lets Jenkins offload jobs from the master.

![](https://github.com/Gagandeep70762/staticdeploy-ci-cd-jenkins/blob/main/images/17.png) 

---

## 🔧 6 - Test Connection with Simple Freestyle Job

- Go to Jenkins Dashboard → New Item → Freestyle project
- Name it `test-connection`
- In **Build Environment**, restrict it to the label of your slave node (e.g., `slave1`)
- Under **Build**, add a simple shell step:
  ```bash
  echo "The Jenkin agent is connected"
  ```
- Save and run the job

✅ You should see output confirming the job executed on the slave.

![](https://github.com/Gagandeep70762/staticdeploy-ci-cd-jenkins/blob/main/images/18.png)

---

🌐 7 - GitHub Integration via Webhook

- Go to GitHub → Repository Settings → Webhooks → Add Webhook
  - Payload URL: `http://<jenkins-ip>:8080/github-webhook/`
  - Content type: `application/json`
  - Select "Just the push event"
  - Click **Add webhook**
![](https://github.com/Gagandeep70762/staticdeploy-ci-cd-jenkins/blob/main/images/4.png)

✅ Now every code push triggers Jenkins automatically!

![](https://github.com/Gagandeep70762/staticdeploy-ci-cd-jenkins/blob/main/images/20.png)
![](https://github.com/Gagandeep70762/staticdeploy-ci-cd-jenkins/blob/main/images/21.png)

---

## 📊 8 - Real-Time Code Trigger Verification

- Edit your index.html in GitHub (e.g., change title)
- Commit and push
- Check Jenkins console → new build should trigger and reflect changes

![](https://github.com/Gagandeep70762/staticdeploy-ci-cd-jenkins/blob/main/images/3.png)

---

## 💻 9 - Configure Slave1 EC2 Instance and Agent Connection

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

---

## 🔌 10 - Add Node Entry for Slave1 in Jenkins

- In Jenkins → Manage Jenkins → Nodes → New Node
- Enter: `slave1`, choose Permanent Agent
- Add SSH credentials, use same Public IP and port

- ✨ Now repeat the same steps to create another EC2 instance named `Jenkins Slave2`
- Use identical configuration as `Slave1` (Amazon Linux 2, Docker, Java, and SSH access)
- Add this as `slave2` in Jenkins → Manage Jenkins → Nodes → New Node
- This helps distribute Jenkins jobs and manage build traffic efficiently

![](https://github.com/Gagandeep70762/staticdeploy-ci-cd-jenkins/blob/main/images/4.png)

---

## 🔧 11 - Create GitHub Integration Pipeline

- Create new Jenkins Pipeline job: `GitHub-CI-Test`
- Add GitHub Repository URL under Source Code Management (SCM)
- Use `main` branch
- Add pipeline script with Docker build steps
- Save and build → Green success tick

---

![](https://github.com/Gagandeep70762/staticdeploy-ci-cd-jenkins/blob/main/images/5.jpg)

---

## ⚙️ 12 - Final DevOps Pipeline Execution (Production Build)

- Create pipeline: `DevOps-Production`
- Add same GitHub repository & credentials

```bash
docker build -t gagan .
docker run -d -p 85:80 gagan
```

![](https://github.com/Gagandeep70762/staticdeploy-ci-cd-jenkins/blob/main/images/22.png)

![](https://github.com/Gagandeep70762/staticdeploy-ci-cd-jenkins/blob/main/images/23.png)

- Add Jenkinsfile script with Docker image build:

![](https://github.com/Gagandeep70762/staticdeploy-ci-cd-jenkins/blob/main/images/24.png)

---

## 🌍 13 - Deploy and Access Static Website

- Go to AWS EC2 → Jenkins Slave1 → Copy Public IP
- Edit Security Group → Inbound Rules → Add Custom TCP: Port 85

![](https://github.com/Gagandeep70762/staticdeploy-ci-cd-jenkins/blob/main/images/26.png)
- Open browser → Visit: `http://<slave1_ip>:85`

![](https://github.com/Gagandeep70762/staticdeploy-ci-cd-jenkins/blob/main/images/25.png)

- Static Website should now be live
- ![](https://github.com/Gagandeep70762/staticdeploy-ci-cd-jenkins/blob/main/images/27.png)

---

## ✏️ 14 - Real-time Code Update and Reflect

- Go to GitHub repo → Edit HTML (e.g. Change title "shamrock" to "convent")
- Commit changes → Jenkins webhook triggers new build
- Website instantly reflects updated content

![](https://github.com/Gagandeep70762/staticdeploy-ci-cd-jenkins/blob/main/images/34.png)
Updated :
![](https://github.com/Gagandeep70762/staticdeploy-ci-cd-jenkins/blob/main/images/33.png)

---

## 🌟 Final Highlights

- ✅ End-to-end CI/CD pipeline built using **Jenkins**, **GitHub**, **Docker**, and **EC2**
- ♻️ Real-time updates with **GitHub Webhooks** triggering Jenkins builds
- 🚀 Static site deployment on **EC2 Slave1** with port 85 exposed
- 💻 Confirmed content updates pushed to GitHub reflected instantly live

---

## 🔮 Future Steps

- ☁️ Integrate **S3 & CloudFront** for production-level static hosting
- 🔐 Configure **SSL with Let's Encrypt** and reverse proxy using **NGINX**
- 📊 Add monitoring with **Prometheus & Grafana**
- 🛡️ Harden Jenkins setup with **IAM roles** and **backup via S3/EBS**
