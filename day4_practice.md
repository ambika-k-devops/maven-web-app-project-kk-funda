# 🚀 Nexus Repository Manager Setup & Jenkins Integration Guide

> Complete step-by-step guide for installing Nexus Repository Manager on AWS EC2 (RHEL), configuring repositories, integrating with Jenkins, and deploying Maven artifacts.

## Architecture

```text
Developer
    |
    v
GitHub
    |
    v
Jenkins
    |
    v
Maven Build
    |
    v
SonarQube Analysis
    |
    v
Nexus Repository
    |
    v
Tomcat / Docker / Kubernetes
```

---

# Prerequisites

- AWS EC2: **t2.medium** (4 GB RAM)
- Ports:
  - 22 (SSH)
  - 8081 (Nexus)
- RHEL / Amazon Linux
- Internet connectivity

> **Note**
>
> Nexus is distributed as a `.tar.gz` archive. It is not installed using an installer.

---

# Install Nexus

## 1. Connect to EC2

```bash
ssh -i key.pem ec2-user@<PUBLIC-IP>
sudo su -
```

## 2. Verify Memory

```bash
free -h
```

## 3. Install Required Packages

```bash
yum install tar wget tree -y
```

## 4. Move to /opt

```bash
cd /opt
```

## 5. Download Nexus

```bash
wget https://download.sonatype.com/nexus/3/nexus-3.92.3-01-linux-x86_64.tar.gz
```

## 6. Extract

```bash
tar -xzvf nexus-3.92.3-01-linux-x86_64.tar.gz
```

Creates:

```text
nexus-3.92.3-01/
sonatype-work/
```

## 7. Rename Directory

```bash
mv /opt/nexus-3.92.3-01 /opt/nexus
```

## 8. Create Nexus User

```bash
useradd nexus
```

## 9. Give Sudo Permission

```bash
visudo
```

Add:

```text
nexus ALL=(ALL) NOPASSWD: ALL
```

## 10. Change Ownership

```bash
chown -R nexus:nexus /opt/nexus
chown -R nexus:nexus /opt/sonatype-work
```

## 11. Set Permissions

```bash
chmod -R 775 /opt/nexus
chmod -R 775 /opt/sonatype-work
```

## 12. Configure nexus.rc

```bash
vi /opt/nexus/bin/nexus.rc
```

Update:

```bash
run_as_user="nexus"
```

---

# Configure Systemd Service

Create:

```bash
vi /etc/systemd/system/nexus.service
```

```ini
[Unit]
Description=Nexus Repository Manager
After=network.target

[Service]
Type=forking
LimitNOFILE=65536
ExecStart=/opt/nexus/bin/nexus start
ExecStop=/opt/nexus/bin/nexus stop
User=nexus
Restart=on-abort

[Install]
WantedBy=multi-user.target
```

Reload:

```bash
systemctl daemon-reexec
systemctl daemon-reload
```

Enable and Start:

```bash
systemctl enable nexus
systemctl start nexus
systemctl status nexus
```

---

# Access Nexus

```
http://<PUBLIC-IP>:8081
```

Initial password:

```bash
cat /opt/sonatype-work/nexus3/admin.password
```

Login:

- Username: `admin`
- Password: output from `admin.password`

---

# Create Repositories

Create two Maven Hosted repositories.

## Release

- Name: `jio-release`
- Version Policy: Release

## Snapshot

- Name: `jio-snapshot`
- Version Policy: Snapshot

---

# Jenkins Integration

## Step 1

Connect to the Maven project.

Verify `pom.xml` exists.

## Step 2 - Configure Repository URLs

Update `pom.xml`.

```xml
<distributionManagement>

    <repository>
        <id>nexus</id>
        <name>KK FUNDA Releases Nexus Repository</name>
        <url>http://<NEXUS-IP>:8081/repository/jio-release/</url>
    </repository>

    <snapshotRepository>
        <id>nexus</id>
        <name>KK FUNDA Snapshot Nexus Repository</name>
        <url>http://<NEXUS-IP>:8081/repository/jio-snapshot/</url>
    </snapshotRepository>

</distributionManagement>
```

---

# Repository Details vs Credentials

| Configuration | Location |
|--------------|----------|
| Repository URLs | pom.xml |
| Nexus Credentials | settings.xml |
| Maven Goals | Jenkins Job |

---

# Locate settings.xml

```bash
find / -name settings.xml
```

Example:

```text
/var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/maven_3.9.9/conf/settings.xml
```

Open:

```bash
vi /var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/maven_3.9.9/conf/settings.xml
```

> **Important:** here `/tools` folder will be auto generate only when first build triggered in jenkins job.

> **Important:** maven version user `maven-3.9.16`


Add inside `<servers>`:

```xml
<server>
    <id>nexus</id>
    <username>admin</username>
    <password>password</password>
</server>
```

> **Important:** `<id>` in `settings.xml` **must exactly match** the `<id>` in `pom.xml`.

---

# Jenkins Job

Navigate:

```
Job
└── Configure
      └── Build
            └── Invoke Top Level Maven Targets
```

Use this Maven Goal:

```bash
clean sonar:sonar -Dsonar.token=<SONAR_TOKEN> deploy
```

Example:

```bash
clean sonar:sonar -Dsonar.token=squ_xxxxxxxxxxxxxxxxx deploy
```

> Maven executes goals from **left to right**.
>
> Always keep **deploy** as the **last goal**.

Correct:

```bash
clean sonar:sonar deploy
```

Wrong:

```bash
clean deploy sonar:sonar
```

Save the job and click **Build Now**.

---

# Verify Artifact Upload

Browse:

```
Nexus
└── Browse
      ├── jio-release
      └── jio-snapshot
```

Expected:

```
groupId
 └── artifactId
      └── version
           ├── jar
           ├── war
           └── pom
```

---

# Troubleshooting

## wget not found

```bash
yum install wget -y
```

## Nexus not starting

```bash
tail -50 /opt/sonatype-work/nexus3/log/nexus.log
```

Look for:

```
Started Sonatype Nexus COMMUNITY
```

## Artifact not uploading

Verify:

- Repository URL in `pom.xml`
- Credentials in `settings.xml`
- `<id>nexus</id>` matches in both files
- `deploy` is the last Maven goal

---

# Useful Commands

```bash
systemctl start nexus
systemctl stop nexus
systemctl restart nexus
systemctl status nexus

tail -f /opt/sonatype-work/nexus3/log/nexus.log

ss -tulpn | grep 8081
```

---

# DevOps Interview Questions

### What is Nexus?

An artifact repository used to store and manage build artifacts.

### Default Port

`8081`

### Where are repository URLs configured?

`pom.xml`

### Where are credentials configured?

`settings.xml`

### Which Maven goal uploads artifacts?

`deploy`

### Release vs Snapshot

| Release | Snapshot |
|----------|----------|
| Immutable | Mutable |
| Production | Development |

---

# Author

**Ambika Rama Katyayini Kachetti**

DevOps | AWS | Jenkins | Maven | SonarQube | Nexus | Docker | Kubernetes
