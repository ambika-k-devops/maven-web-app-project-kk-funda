# Jenkins, GitHub & Maven Integration on AWS EC2

## Objective

To set up Jenkins on an AWS EC2 instance, integrate it with a GitHub repository, and perform Maven-based build automation.

---

## Environment

* AWS EC2 (Red Hat Linux)
* Jenkins
* GitHub
* Maven
* Java 21
* MobaXterm (SSH Client)

---

## Implementation Steps

### Step 1: Launch EC2 Instance

* Create a Red Hat EC2 instance (t2.medium).
* Note the Public IP address.
* Download the PEM key file.

### Step 2: Connect to EC2

Using MobaXterm:

* Host: EC2 Public IP
* User: ec2-user
* Authentication: PEM Key

Switch to root user:

```bash
sudo su -
```

---

### Step 3: Install Jenkins

Add Jenkins repository:

```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo \
https://pkg.jenkins.io/rpm-stable/jenkins.repo
```

Update packages:

```bash
sudo yum upgrade
```

Install Java:

```bash
sudo yum install fontconfig java-21-openjdk
```

Install Jenkins:

```bash
sudo yum install jenkins
```

Reload system services:

```bash
sudo systemctl daemon-reload
```

---

### Step 4: Install Additional Utilities

```bash
yum install wget tree -y
```

Verify Jenkins installation:

```bash
rpm -qa | grep jenkins
```

or

```bash
which jenkins
```

---

### Step 5: Start Jenkins Service

Enable Jenkins service:

```bash
sudo systemctl enable jenkins
```

Check status:

```bash
sudo systemctl status jenkins
```

Start Jenkins:

```bash
sudo systemctl start jenkins
```

Verify service status:

```bash
sudo systemctl status jenkins
```

---

### Step 6: Configure Security Group

Add inbound rule:

| Type       | Port | Source        |
| ---------- | ---- | ------------- |
| Custom TCP | 8080 | Anywhere IPv4 |

---

### Step 7: Access Jenkins Dashboard

Open browser:

```text
http://<EC2-Public-IP>:8080
```

Retrieve Jenkins initial password:

```bash
cat /var/lib/jenkins/secrets/initialAdminPassword
```

or

```bash
vi /var/lib/jenkins/secrets/initialAdminPassword
```

Paste the password into Jenkins setup page.

---

### Step 8: Complete Jenkins Setup

* Install Suggested Plugins
* Create Admin User

Example:

* Username: Ambika_DevOps_Engineer
* Full Name: Ambika
* Email: ambika9779

---

### Step 9: Create Jenkins Freestyle Project

Project Name:

```text
Ambika_DevOps_Jenkins
```

---

### Step 10: Integrate GitHub Repository

Repository:

```text
https://github.com/AmbikaKachetti/maven-web-app-project.git
```

Branch:

```text
development
```

---

### Issue Encountered

Error:

```text
Failed to connect to repository:
Error performing git command:
git ls-remote -h
```

### Resolution

Git was not installed on the Jenkins server.

Install Git:

```bash
yum install git -y
```

After installation, Jenkins successfully cloned the repository from the development branch.

---

### Step 11: Configure Maven Build

Navigate to:

```text
Manage Jenkins → Global Tool Configuration
```

Configure Maven installation.

Build Goal:

```text
clean package
```

Run the build.

<img width="698" height="608" alt="Screenshot 2026-06-23 023152" src="https://github.com/user-attachments/assets/d8cb8a39-3e8a-492b-8151-29d7694f25e9" />
#### build after maven
<img width="1320" height="621" alt="Screenshot 2026-06-23 023304" src="https://github.com/user-attachments/assets/cafa6cb0-3bef-4c27-9210-8980ff557c59" />

##### MobaXterm - before build and after build results of "ls -lrth" in Ambika_DevOps_Jenkins job
<img width="874" height="627" alt="Screenshot 2026-06-23 023427" src="https://github.com/user-attachments/assets/a2a550a1-8f1e-4769-b2a8-db4a952ce120" />

---

## Results

✅ Jenkins installed successfully

✅ GitHub repository integrated successfully

✅ Git installed and configured

✅ Maven integrated with Jenkins

✅ Source code cloned from GitHub

✅ Maven build completed successfully

✅ Basic CI/CD workflow implemented

---

## Skills Practiced

* AWS EC2
* Linux Administration
* Git
* GitHub
* Jenkins
* Maven
* Java
* Build Automation
* Continuous Integration (CI)
* Troubleshooting Jenkins Issues
