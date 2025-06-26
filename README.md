
# 🚀 Jenkins CI/CD on AWS EC2 - Static Site Deployment (Video-Based)

This guide walks you through building a CI/CD pipeline on **Amazon EC2 (Amazon Linux)** using Jenkins, based on the steps shown in the referenced video.

---

## ✅ Step-by-Step Setup with Screenshots

### 🔹 0.1 - Launch Jenkins Master EC2 Instance
Launch an Amazon Linux EC2 instance to host Jenkins Master. Use default ports 22 and 8080 for SSH and Jenkins access.

![Launch EC2](images/step-0.1-launch-ec2-instance.png)

---

### 🔹 0.2 - Add User Data to Install Java & Jenkins Automatically
During instance launch, go to **Advanced → User Data** and add the shell script to install Jenkins.

![User Data](images/step-0.2-user-data-install-jenkins.png)

---

### 🔹 1 - Access Jenkins via Public IP and Port 8080
Once Jenkins is running, open your browser and go to:

```
http://<EC2-PUBLIC-IP>:8080
```

You will be prompted to unlock Jenkins.

![Access Jenkins](images/step-1-access-jenkins-8080.png)

---

### 🔹 2 - Unlock Jenkins Using Initial Password
SSH into the EC2 instance and run:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Paste the password into the browser.

![Unlock Jenkins](images/step-2-unlock-jenkins.png)

---

### 🔹 3 - Install Suggested Jenkins Plugins
Select **Install Suggested Plugins** to get basic Jenkins functionality ready.

![Install Plugins](images/step-3-install-plugins.png)

---

### 🔹 4 - Create Slave 1 and Allow Port 85 in Security Group
- Create a new EC2 instance as **Slave 1**.
- Edit its Security Group to **allow port 85** (for serving web content).
- Configure the slave node from Jenkins UI.

![Create Slave1 & Allow Port 85](images/step-4-create-slave1-port85.png)

---

### 🔹 5 - Build Pipeline to Deploy Static Website via Slave1
- Create a Jenkins pipeline.
- The pipeline runs a Python HTTP server on port 85.
- Access the live website:

```
http://<Slave1-PUBLIC-IP>:85
```

![Verify Site on Port 85](images/step-5-verify-pipeline-site-on-port85.png)

---

## 🔁 GitHub Integration & Live Update

- Push code to GitHub → triggers webhook to Jenkins
- Pipeline runs again → website updates instantly

---

## 📌 Final Notes

- Jenkins Master handles orchestration
- Slave1 builds and serves website
- GitHub triggers jobs automatically
- Pipeline demonstrates end-to-end CI/CD with instant updates

---

⏳ Let me know when you're ready to upload images for each step (0.1 to 5), and I’ll regenerate the final README with them embedded.
