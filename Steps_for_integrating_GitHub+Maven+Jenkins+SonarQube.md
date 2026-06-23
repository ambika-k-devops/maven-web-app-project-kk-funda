# GitHub + Maven + Jenkins + SonarQube Integration

## Project Overview

This project demonstrates the integration of GitHub, Maven, Jenkins, and SonarQube to implement a basic Continuous Integration (CI) pipeline.

### Tools Used

* Git & GitHub
* Maven
* Jenkins
* SonarQube
* AWS EC2 (Red Hat Linux)
* Java 21

---

# SonarQube Installation on AWS EC2

## Step 1: Launch EC2 Instance

Launch a Red Hat EC2 instance with:

* Instance Type: t2.medium
* RAM: 4 GB

---

## Step 2: Connect to EC2 Server

Connect using MobaXterm or SSH.

```bash
ssh -i <pem-file> ec2-user@<public-ip>
```

Switch to root user:

```bash
sudo su -
```

---

## Step 3: Install Java

```bash
yum install java-21-openjdk-devel -y
```

Verify installation:

```bash
javac --version
```

---

## Step 4: Download SonarQube

Move to the /opt directory:

```bash
cd /opt
```

Install required utilities:

```bash
yum install wget unzip tree -y
```

Download SonarQube:

```bash
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-26.6.0.123539.zip
```

Extract package:

```bash
unzip sonarqube-26.6.0.123539.zip
```

Rename directory:

```bash
mv sonarqube-26.6.0.123539 sonarqube
```

---

## Step 5: Create SonarQube User

For security reasons, SonarQube should not run as the root user.

Create a dedicated user:

```bash
useradd sonar
```

Grant sudo access:

```bash
visudo
```

Add:

```text
sonar ALL=(ALL) NOPASSWD: ALL
```

---

## Step 6: Set Permissions

```bash
chown -R sonar:sonar /opt/sonarqube/
chmod -R 775 /opt/sonarqube/
```

Switch to sonar user:

```bash
su - sonar
```

---

## Step 7: Start SonarQube

```bash
cd /opt/sonarqube/bin/linux-x86-64
```

Start SonarQube:

```bash
sh sonar.sh start
```

Check status:

```bash
sh sonar.sh status
```

---

## Step 8: Access SonarQube

Default Port:

```text
9000
```

Open browser:

```text
http://<EC2-PUBLIC-IP>:9000
```

Ensure port 9000 is allowed in the EC2 Security Group.

---

## Step 9: Login to SonarQube

Default Credentials:

```text
Username: admin
Password: admin
```

Change the password after first login.

---

# Jenkins Integration with SonarQube

## Initial Build Command

```bash
clean package sonar:sonar
```

### Error Encountered

Build failed with authentication error.

#### Error Screenshot build failed due to issues in pom.xml

<img width="1321" height="627" alt="SonarQube Authentication Error" src="https://github.com/user-attachments/assets/016df60f-746d-432d-9559-fe5c6c933e9c" />

---

## Fix 1: Update pom.xml

Remove:

```xml
<sonar.login>admin</sonar.login>
<sonar.password>password</sonar.password>
```

Add:

```xml
<sonar.projectKey>maven-web-application-kkfunda</sonar.projectKey>
<sonar.projectName>maven-web-application-kkfunda</sonar.projectName>
```

Update SonarQube URL:

```xml
<sonar.host.url>http://<SONARQUBE-IP>:9000</sonar.host.url>
```

---

## Fix 2: Generate SonarQube Token

Navigate to:

```text
My Account → Security → Generate Token
```

Token Name:

```text
Jenkins-Token
```

#### Token Generation Screenshot

<img width="1344" height="406" alt="Screenshot 2026-06-24 014337" src="https://github.com/user-attachments/assets/e999b6d8-d3a2-4d32-8dba-cbf911bd66d5" />

Copy the generated token.

---

## Fix 3: Update Jenkins Build Goal

Replace:

```bash
clean package sonar:sonar
```

With:

```bash
clean package sonar:sonar -Dsonar.token=YOUR_GENERATED_TOKEN
```

---

## Build Verification

Run the Jenkins job again.

Expected Result:

* Source Code Checkout Successful
* Maven Build Successful
* SonarQube Analysis Successful
* Jenkins Build Status: SUCCESS

---

# CI Pipeline Flow

```text
GitHub
   ↓
Jenkins
   ↓
Maven Build
   ↓
SonarQube Analysis
   ↓
Build Success
```
Build is from github development branch in this repo: " https://github.com/ambika-k-devops/maven-web-app-project-kk-funda.git "
<img width="1360" height="587" alt="Screenshot 2026-06-24 014800" src="https://github.com/user-attachments/assets/aca91b7d-3c67-4178-ace3-052f21e2eeea" />
Build Success after correcting pom.xml
<img width="1321" height="610" alt="Screenshot 2026-06-24 012538" src="https://github.com/user-attachments/assets/c527ec9f-1212-4981-94a6-d82e46b90565" />
code quality report got created in sonarqube
<img width="1348" height="549" alt="Screenshot 2026-06-24 014033" src="https://github.com/user-attachments/assets/5792178a-0c64-44df-a92f-8b2135f2adb1" />
<img width="1335" height="593" alt="Screenshot 2026-06-24 010304" src="https://github.com/user-attachments/assets/48fdb719-c94b-4c98-8faf-e5f418023843" />

---

# Skills Practiced

* AWS EC2
* Linux Administration
* Git
* GitHub
* Maven
* Jenkins
* SonarQube
* Java
* Continuous Integration (CI)
* Build Automation
* Static Code Analysis
* Troubleshooting Build Failures

---

# Project Status

✅ GitHub Integration Completed

✅ Maven Integration Completed

✅ Jenkins Integration Completed

✅ SonarQube Integration Completed

✅ CI Pipeline Working Successfully
