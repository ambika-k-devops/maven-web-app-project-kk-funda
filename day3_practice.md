# 🚀 Nexus Repository Manager Installation on AWS EC2

## 📌 Project Overview

This project demonstrates the installation and configuration of **Nexus Repository Manager 3.92.3-01 Community Edition** on an AWS EC2 Red Hat Linux server.

Nexus is used to store and manage build artifacts such as JAR and WAR files generated during CI/CD pipelines.

### Tools Used

* AWS EC2 (Red Hat Linux)
* Java 21
* Nexus Repository Manager 3
* Linux

---

# 🔧 Nexus Installation Steps

## Step 1: Switch to Root User

```bash
sudo su -
```

---

## Step 2: Move to /opt Directory

```bash
cd /opt
```

Verify downloaded package:

```bash
yum install tar wget tree -y
```

```bash
wget https://download.sonatype.com/nexus/3/nexus-3.92.3-01-linux-x86_64.tar.gz
```

```bash
ls -lrth
```

Expected:

```text
nexus-3.92.3-01-linux-x86_64.tar.gz
```

---

## Step 3: Extract Nexus Package

```bash
tar -xzvf nexus-3.92.3-01-linux-x86_64.tar.gz
```

Rename extracted folder:

```bash
mv /opt/nexus-3.92.3-01 /opt/nexus
```

Verify:

```bash
ls -lrth
```

Expected:

```text
nexus
sonatype-work
```

---

## Step 4: Create Nexus User

For security reasons, Nexus should not run as the root user.

```bash
useradd nexus
```

Grant sudo access:

```bash
visudo
```

Add:

```text
nexus ALL=(ALL) NOPASSWD: ALL
```

---

## Step 5: Assign Permissions

```bash
chown -R nexus:nexus /opt/nexus
chown -R nexus:nexus /opt/sonatype-work

chmod -R 775 /opt/nexus
chmod -R 775 /opt/sonatype-work
```

---

## Step 6: Configure Nexus Runtime User

Edit:

```bash
vi /opt/nexus/bin/nexus.rc
```

Add:

```text
run_as_user="nexus"
```

Save and exit.

---

# ⚠️ Issue Faced During Setup

### Attempted Command

```bash
ln -s /opt/nexus/bin/nexus /etc/init.d/nexus
```

### Error

```text
ln: failed to create symbolic link '/etc/init.d/nexus':
No such file or directory
```

### Reason

Modern Red Hat / Amazon Linux systems use **systemd** instead of **init.d**.

Therefore, this step is not required for manually starting Nexus.

---

# 🚀 Starting Nexus

Switch to nexus user:

```bash
su - nexus
```

Move to Nexus bin directory:

```bash
cd /opt/nexus/bin
```

Verify files:

```bash
ls -l
```

Expected:

```text
nexus
nexus.rc
nexus.vmoptions
sonatype-nexus-repository-3.92.3-01.jar
```

Start Nexus:

```bash
./nexus start
```

Output:

```text
Starting nexus
```

---

# 🔍 Verify Nexus Startup

Check logs:

```bash
cd /opt/sonatype-work/nexus3/log
```

View latest logs:

```bash
tail -50 nexus.log
```

Important success messages:

```text
Started ServerConnector ... {0.0.0.0:8081}

Started Sonatype Nexus COMMUNITY 3.92.3-01
```

These messages confirm Nexus started successfully.

---

# 🔌 Verify Nexus Port

Check if Nexus is listening on port 8081:

```bash
ss -tulnp | grep 8081
```

Output:

```text
tcp LISTEN *:8081
```

This confirms Nexus is running successfully.

---

# 🌐 Access Nexus UI

Enable port **8081** in the EC2 Security Group.

Open browser:

```text
http://<EC2-PUBLIC-IP>:8081
```

Example:

```text
http://15.207.110.48:8081
```

---

# 🔑 Get Initial Admin Password

```bash
cat /opt/sonatype-work/nexus3/admin.password
```

Example:

```text
c8c18846-b4ee-429d-9975-72e0343da053
```

Use:

```text
Username : admin
Password : <generated password>
```

---

# 🔒 Change Admin Password

After first login:

```text
New Password : admin@123
Confirm Password : admin@123
```

---

# 📚 Skills Practiced

* AWS EC2
* Linux Administration
* Java 21
* Nexus Repository Manager
* Artifact Repository Management
* User & Permission Management
* Service Troubleshooting
* Log Analysis

---

# 🎯 Outcome

✅ Nexus Repository Manager Installed Successfully

✅ Nexus Running on Port 8081

✅ Admin Access Configured

✅ Artifact Repository Ready for CI/CD Integration

### Next Integration

```text
GitHub
   ↓
Jenkins
   ↓
Maven
   ↓
SonarQube
   ↓
Nexus
   ↓
Docker
   ↓
Tomcat
```
AWS Nexus Instance: t2.medium
<img width="1357" height="235" alt="Screenshot 2026-06-24 224546" src="https://github.com/user-attachments/assets/a90977b1-77bb-412f-a657-dd54d759fb83" />

Secutiry Group -> inbound rule for 8081 port enabled for nexus
<img width="1350" height="460" alt="Screenshot 2026-06-24 224515" src="https://github.com/user-attachments/assets/846a9ab0-f495-476b-9c80-965adb8e18a1" />

Nexus Interface
<img width="1339" height="674" alt="Screenshot 2026-06-24 224450" src="https://github.com/user-attachments/assets/b7bf30ed-b987-4a78-bf1a-521c14903356" />

