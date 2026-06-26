# 🚀 Apache Tomcat 9.0.118 Installation & Jenkins Integration Guide

### Part 1 – Apache Tomcat Installation on RHEL / Amazon Linux

<p align="center">

![Tomcat](https://img.shields.io/badge/Apache-Tomcat%209.0.118-F8DC75?style=for-the-badge\&logo=apachetomcat\&logoColor=black)
![Java](https://img.shields.io/badge/Java-21-red?style=for-the-badge\&logo=openjdk)
![Platform](https://img.shields.io/badge/Platform-RHEL%20%7C%20Amazon%20Linux-blue?style=for-the-badge\&logo=redhat)
![Jenkins](https://img.shields.io/badge/Jenkins-Integration-D24939?style=for-the-badge\&logo=jenkins)

</p>

---

# 📖 Table of Contents

* [1. Introduction](#1-introduction)
* [2. Architecture](#2-architecture)
* [3. Prerequisites](#3-prerequisites)
* [4. Launch EC2 Instance](#4-launch-ec2-instance)
* [5. Connect to EC2](#5-connect-to-ec2)
* [6. Become Root User](#6-become-root-user)
* [7. Install Java 21](#7-install-java-21)
* [8. Install Required Packages](#8-install-required-packages)
* [9. Download Apache Tomcat](#9-download-apache-tomcat)
* [10. Extract Apache Tomcat](#10-extract-apache-tomcat)
* [11. Understand Tomcat Directory Structure](#11-understand-tomcat-directory-structure)
* [12. Make Startup Scripts Executable](#12-make-startup-scripts-executable)
* [13. Start Tomcat](#13-start-tomcat)
* [14. Verify Tomcat](#14-verify-tomcat)
* [15. Configure AWS Security Group](#15-configure-aws-security-group)
* [16. Configure Firewall (Optional)](#16-configure-firewall-optional)
* [17. Useful Commands](#17-useful-commands)

<!-- Continue with the remaining sections... -->

---

# 1. Introduction

Apache Tomcat is an **open-source Java Servlet Container** developed by the Apache Software Foundation.

Tomcat is used to deploy and run:

* Java Servlets
* JSP (Java Server Pages)
* Java Web Applications (`.war` files)

In a DevOps CI/CD pipeline:

```text
Developer
    │
    ▼
GitHub
    │
    ▼
Jenkins
    │
    ▼
Maven
    │
    ▼
Tomcat
    │
    ▼
Browser
```

---

> [!NOTE]
>
> Tomcat is **not** a full Java EE Application Server like WebLogic or JBoss.
>
> It is a lightweight Servlet Container primarily used for Java web applications.

---

# 2. Architecture

```text
                   Developer

                       │
                  git push

                       │

                       ▼

                GitHub Repository

                       │

                       ▼

                  Jenkins Server

                       │

                Maven Build (.war)

                       │

                       ▼

                Apache Tomcat

                       │

                       ▼

                  End Users
```

---

# 3. Prerequisites

## EC2 Requirements

| Resource      | Recommendation             |
| ------------- | -------------------------- |
| OS            | RHEL 9 / Amazon Linux 2023 |
| Instance Type | t2.micro / t2.small        |
| RAM           | Minimum 1 GB               |
| Storage       | 20 GB                      |
| Java          | OpenJDK 21                 |

---

## Required Ports

| Port | Purpose |
| ---- | ------- |
| 22   | SSH     |
| 8080 | Tomcat  |

---

## Java Compatibility

| Tomcat Version | Minimum Java |
| -------------- | ------------ |
| 9.0.x          | Java 8       |
| Recommended    | Java 21      |

---

> [!TIP]
>
> Since you already installed **Java 21**, no additional Java version is required.

---

# 4. Launch EC2 Instance

Launch a new EC2 instance.

### Select

* RHEL 9
* Amazon Linux 2023

Recommended:

```text
Instance Type

t2.micro
```

Allow inbound rules:

```text
SSH      22

Custom TCP

8080
```

---

# 5. Connect to EC2

SSH into your server.

```bash
ssh -i my-key.pem ec2-user@<PUBLIC-IP>
```

Example

```bash
ssh -i devops.pem ec2-user@43.205.xxx.xxx
```

---

## Verify

```bash
hostname
```

---

# 6. Become Root User

Although Tomcat can run as a non-root user, for this lab we'll use the root account.

```bash
sudo su -
```

Verify

```bash
whoami
```

Expected

```text
root
```

---

# 7. Install Java 21

## Install Java Runtime + Development Kit

```bash
yum install java-21-openjdk java-21-openjdk-devel fontconfig -y
```

---

### Why these packages?

| Package               | Purpose                         |
| --------------------- | ------------------------------- |
| java-21-openjdk       | Java Runtime                    |
| java-21-openjdk-devel | Java Compiler (`javac`)         |
| fontconfig            | Required by some Java libraries |

---

## Verify Java

```bash
java -version
```

Expected

```text
openjdk version "21"
```

Verify Compiler

```bash
javac -version
```

Expected

```text
javac 21.x.x
```

---

> [!IMPORTANT]
>
> If `java` or `javac` commands are not found, Java installation failed. Do not continue until this is resolved.

---

# 8. Install Required Packages

Install commonly used Linux utilities.

```bash
yum install wget tar unzip tree lsof -y
```

---

## Package Explanation

| Package | Purpose                  |
| ------- | ------------------------ |
| wget    | Download files           |
| tar     | Extract `.tar.gz`        |
| unzip   | Extract ZIP archives     |
| tree    | View directory structure |
| lsof    | Check listening ports    |

---

Verify

```bash
tree --version
```

---

# 9. Download Apache Tomcat

Move to the `/opt` directory.

```bash
cd /opt
```

---

Download Tomcat.

```bash
wget https://downloads.apache.org/tomcat/tomcat-9/v9.0.118/bin/apache-tomcat-9.0.118.tar.gz
```

---

Verify

```bash
ls
```

Expected

```text
apache-tomcat-9.0.118.tar.gz
```

---

> [!NOTE]
>
> We use the **`.tar.gz`** package on Linux because it is the standard distribution format for Unix/Linux systems. It preserves file permissions better than the ZIP package.

---

# 10. Extract Apache Tomcat

Extract the archive.

```bash
tar -xvzf apache-tomcat-9.0.118.tar.gz
```

---

Verify

```bash
ls
```

Expected

```text
apache-tomcat-9.0.118
```

---

Move inside.

```bash
cd apache-tomcat-9.0.118
```

Verify

```bash
pwd
```

Expected

```text
/opt/apache-tomcat-9.0.118
```

---

# 11. Understand Tomcat Directory Structure

Run

```bash
tree -L 1
```

Expected

```text
apache-tomcat-9.0.118

├── bin
├── conf
├── lib
├── logs
├── temp
├── webapps
├── work
├── LICENSE
└── NOTICE
```

---

## Directory Explanation

| Directory | Description                    |
| --------- | ------------------------------ |
| bin       | Startup and shutdown scripts   |
| conf      | Configuration files            |
| logs      | Tomcat log files               |
| webapps   | Applications are deployed here |
| temp      | Temporary files                |
| work      | Compiled JSP files             |
| lib       | Tomcat libraries               |

---

> [!TIP]
>
> Spend time understanding this directory structure. During interviews, you're often asked where Tomcat stores configuration files, logs, and deployed applications.

---

# 12. Make Startup Scripts Executable

Move into the `bin` directory.

```bash
cd bin
```

Grant execute permissions.

```bash
chmod +x *.sh
```

Verify

```bash
ls -l
```

You should see executable (`x`) permissions on the shell scripts.

---

> [!IMPORTANT]
>
> Without execute permissions, Tomcat cannot start.

---

# 13. Start Tomcat

Start the server.

```bash
./startup.sh
```

Alternative method:

```bash
./catalina.sh start
```

Expected output:

```text
Using CATALINA_BASE:
Using CATALINA_HOME:
Tomcat started.
```

---

# 14. Verify Tomcat

## Method 1 – Check Process

```bash
ps -ef | grep tomcat
```

Expected:

```text
root     12345  java ...
```

---

## Method 2 – Check Port

```bash
lsof -i :8080
```

Expected:

```text
java LISTEN TCP *:8080
```

---

## Method 3 – Local Test

```bash
curl http://localhost:8080
```

If Tomcat is running, you should receive HTML content from the default Tomcat page.

---

## Method 4 – Browser Test

Open:

```text
http://<PUBLIC-IP>:8080
```

Expected:

```text
Apache Tomcat Welcome Page
```

---

# 15. Configure AWS Security Group

If the browser cannot access Tomcat:

Go to:

```text
AWS Console

↓

EC2

↓

Select Instance

↓

Security

↓

Security Groups

↓

Edit Inbound Rules
```

Add:

| Type       | Port | Source               |
| ---------- | ---- | -------------------- |
| Custom TCP | 8080 | Anywhere (0.0.0.0/0) |

Save the rule and refresh the browser.

---

> [!WARNING]
>
> Opening port **8080** to the internet is acceptable for a learning environment. In production, restrict access using security groups, reverse proxies, or private networking.

---

# 16. Configure Firewall (Optional)

If `firewalld` is running:

Check status:

```bash
systemctl status firewalld
```

If active, allow port 8080:

```bash
firewall-cmd --permanent --add-port=8080/tcp
```

Reload:

```bash
firewall-cmd --reload
```

---

# 17. Useful Commands

## Check Java

```bash
java -version
```

---

## Check Compiler

```bash
javac -version
```

---

## Start Tomcat

```bash
cd /opt/apache-tomcat-9.0.118/bin

./startup.sh
```

---

## Stop Tomcat

```bash
./shutdown.sh
```

---

## Restart Tomcat

```bash
./shutdown.sh

sleep 5

./startup.sh
```

---

## Check Process

```bash
ps -ef | grep tomcat
```

---

## Check Listening Port

```bash
lsof -i :8080
```

---

## View Logs

```bash
tail -f /opt/apache-tomcat-9.0.118/logs/catalina.out
```

---

# ✅ Part 1 Summary

By completing Part 1, you have successfully:

* Installed Java 21
* Installed Apache Tomcat 9.0.118
* Started Tomcat
* Verified the installation
* Opened port 8080
* Accessed the Tomcat welcome page
* Learned the Tomcat directory structure
* Practiced essential administration commands

---

## ▶️ Next Part

**Part 2** will cover:

* Tomcat Manager App
* Host Manager
* `context.xml` configuration
* `tomcat-users.xml`
* Creating Manager users
* Manual WAR deployment
* Viewing logs
* Creating global `startTomcat` / `stopTomcat` commands
* Understanding Tomcat configuration files
* Common mistakes and troubleshooting

_____________________________________________________________________________________________________________________________________________________
# 🚀 Apache Tomcat 9.0.118 Installation & Jenkins Integration Guide

# Part 2 – Tomcat Administration, Manager App & WAR Deployment

> **Prerequisite:** Complete **Part 1** before continuing.

---

# 📖 Table of Contents

* [18. Tomcat Configuration Files](#18-tomcat-configuration-files)
* [19. Tomcat Start, Stop & Restart](#19-tomcat-start-stop--restart)
* [20. Creating Global Commands](#20-creating-global-commands)
* [21. Understanding the Manager Application](#21-understanding-the-manager-application)
* [22. Enable Tomcat Manager App](#22-enable-tomcat-manager-app)
* [23. Configure Tomcat Users](#23-configure-tomcat-users)
* [24. Access the Manager Application](#24-access-the-manager-application)
* [25. Understanding Host Manager](#25-understanding-host-manager)
* [26. Enable Host Manager](#26-enable-host-manager)
* [27. Deploy WAR File Manually](#27-deploy-war-file-manually)
* [28. Verify Deployment](#28-verify-deployment)
* [29. Tomcat Log Files](#29-tomcat-log-files)
* [30. Common Administration Commands](#30-common-administration-commands)
* [31. Common Mistakes](#31-common-mistakes)
* [32. Troubleshooting](#32-troubleshooting)

---

# 18. Tomcat Configuration Files

Go to the configuration directory.

```bash
cd /opt/apache-tomcat-9.0.118/conf
```

List the files.

```bash
ls
```

Expected Output

```text
catalina.properties
context.xml
logging.properties
server.xml
tomcat-users.xml
web.xml
```

---

## Important Configuration Files

| File                | Purpose                           |
| ------------------- | --------------------------------- |
| server.xml          | Main Tomcat server configuration  |
| context.xml         | Application context configuration |
| tomcat-users.xml    | Stores Tomcat users and roles     |
| web.xml             | Default deployment descriptor     |
| logging.properties  | Logging configuration             |
| catalina.properties | Tomcat internal properties        |

---

> [!TIP]
>
> These configuration files are among the most frequently discussed in Tomcat interviews.

---

# 19. Tomcat Start, Stop & Restart

Go to the **bin** directory.

```bash
cd /opt/apache-tomcat-9.0.118/bin
```

---

## Start

```bash
./startup.sh
```

Alternative

```bash
./catalina.sh start
```

---

## Stop

```bash
./shutdown.sh
```

Alternative

```bash
./catalina.sh stop
```

---

## Restart

```bash
./shutdown.sh

sleep 5

./startup.sh
```

---

## Verify

```bash
ps -ef | grep tomcat
```

---

# 20. Creating Global Commands

Instead of navigating to the `bin` directory every time, create symbolic links.

## Create Startup Shortcut

```bash
ln -s /opt/apache-tomcat-9.0.118/bin/startup.sh /usr/bin/startTomcat
```

---

## Create Shutdown Shortcut

```bash
ln -s /opt/apache-tomcat-9.0.118/bin/shutdown.sh /usr/bin/stopTomcat
```

---

## Verify

From any directory, run:

```bash
startTomcat
```

Stop:

```bash
stopTomcat
```

---

> [!NOTE]
>
> A symbolic link (soft link) acts like a shortcut. It allows you to execute the Tomcat startup and shutdown scripts from anywhere on the server.

---

# 21. Understanding the Manager Application

The **Manager Application** is a web interface provided by Tomcat.

It allows administrators to:

* Deploy WAR files
* Undeploy applications
* Start applications
* Stop applications
* Reload applications
* View server status

---

## Why Do We Need It?

Jenkins communicates with the **Tomcat Manager Application** when using the **Deploy to Container** plugin.

Without the Manager Application:

* Jenkins cannot deploy WAR files automatically.
* Remote deployments will fail.

---

# 22. Enable Tomcat Manager App

By default, Tomcat only allows access to the Manager App from **localhost (127.0.0.1)**.

To access it remotely for a lab setup, modify the configuration.

Go to:

```bash
cd /opt/apache-tomcat-9.0.118/webapps/manager/META-INF
```

Open the file:

```bash
vi context.xml
```

Locate the `RemoteAddrValve` configuration. It looks similar to:

```xml
<Valve className="org.apache.catalina.valves.RemoteAddrValve"
       allow="127\.\d+\.\d+\.\d+|::1"/>
```

Comment out or remove the entire `RemoteAddrValve` section.

For example:

```xml
<!--
<Valve className="org.apache.catalina.valves.RemoteAddrValve"
       allow="127\.\d+\.\d+\.\d+|::1"/>
-->
```

Save the file and exit.

Restart Tomcat:

```bash
cd /opt/apache-tomcat-9.0.118/bin

./shutdown.sh

sleep 5

./startup.sh
```

---

> [!WARNING]
>
> Do **not** remove the `RemoteAddrValve` restriction on a production server unless you have other security controls (VPN, reverse proxy, IP allowlists, etc.).

---

# 23. Configure Tomcat Users

Go to the configuration directory:

```bash
cd /opt/apache-tomcat-9.0.118/conf
```

Open:

```bash
vi tomcat-users.xml
```

Before the closing `</tomcat-users>` tag, add the following:

```xml
<role rolename="manager-gui"/>
<role rolename="admin-gui"/>

<user username="ambika"
      password="ambika"
      roles="manager-gui,admin-gui"/>

<user username="rama"
      password="rama"
      roles="manager-gui,admin-gui"/>
```

Save the file.

Restart Tomcat:

```bash
cd /opt/apache-tomcat-9.0.118/bin

./shutdown.sh

sleep 5

./startup.sh
```

---

## Role Explanation

| Role        | Purpose                           |
| ----------- | --------------------------------- |
| manager-gui | Access to the Manager Application |
| admin-gui   | Access to the Host Manager        |

---

> [!IMPORTANT]
>
> Use strong passwords in production. The usernames and passwords above are intended only for learning.

---

# 24. Access the Manager Application

Open the browser:

```text
http://PUBLIC-IP:8080/manager
```

Enter the credentials you configured in `tomcat-users.xml`.

If everything is configured correctly, you should see the **Tomcat Web Application Manager** dashboard.

---

## What Can You Do Here?

* Deploy a WAR file
* Undeploy an application
* Start an application
* Stop an application
* Reload an application
* View server information

---

# 25. Understanding Host Manager

The **Host Manager** is another Tomcat web application used to manage **virtual hosts**.

It is less commonly used than the Manager App, but it is useful when hosting multiple websites on the same Tomcat server.

---

# 26. Enable Host Manager

Go to:

```bash
cd /opt/apache-tomcat-9.0.118/webapps/host-manager/META-INF
```

Open:

```bash
vi context.xml
```

Locate the `RemoteAddrValve` configuration and comment it out, just as you did for the Manager App.

Save the file.

Restart Tomcat.

Open:

```text
http://PUBLIC-IP:8080/host-manager
```

Log in using the same credentials.

---

# 27. Deploy WAR File Manually

Go to the Tomcat deployment directory:

```bash
cd /opt/apache-tomcat-9.0.118/webapps
```

Copy your WAR file into this directory.

Example:

```bash
cp /home/ec2-user/maven-web-application.war .
```

Tomcat automatically detects the WAR file, extracts it, and deploys the application.

---

## Verify

Run:

```bash
ls
```

Example Output:

```text
maven-web-application
maven-web-application.war
ROOT
docs
examples
host-manager
manager
```

The extracted directory indicates that Tomcat successfully deployed the application.

---

# 28. Verify Deployment

Open the application in your browser:

```text
http://PUBLIC-IP:8080/maven-web-application
```

If your application uses a different context path, replace `maven-web-application` with the appropriate name.

---

## Check Logs

If the application doesn't load, inspect the logs:

```bash
tail -f /opt/apache-tomcat-9.0.118/logs/catalina.out
```

---

# 29. Tomcat Log Files

Go to the logs directory:

```bash
cd /opt/apache-tomcat-9.0.118/logs
```

List the files:

```bash
ls
```

Typical log files include:

```text
catalina.out
localhost.log
manager.log
host-manager.log
```

---

## Monitor Logs in Real Time

```bash
tail -f catalina.out
```

This is one of the most common commands used when troubleshooting Tomcat issues.

---

# 30. Common Administration Commands

## Check Running Process

```bash
ps -ef | grep tomcat
```

---

## Check Listening Port

```bash
lsof -i :8080
```

---

## Check Java Version

```bash
java -version
```

---

## Restart Tomcat

```bash
cd /opt/apache-tomcat-9.0.118/bin

./shutdown.sh

sleep 5

./startup.sh
```

---

## View Logs

```bash
tail -f /opt/apache-tomcat-9.0.118/logs/catalina.out
```

---

# 31. Common Mistakes

### ❌ Forgetting to restart Tomcat

Changes to configuration files such as `context.xml` or `tomcat-users.xml` require a restart.

---

### ❌ Editing the wrong file

Ensure you're modifying:

```text
webapps/manager/META-INF/context.xml
```

and

```text
webapps/host-manager/META-INF/context.xml
```

These are different from the global `conf/context.xml`.

---

### ❌ Incorrect XML Syntax

A missing tag or quote in `tomcat-users.xml` can prevent Tomcat from starting.

---

### ❌ Weak Security

Do not expose the Manager App to the public internet in production.

---

### ❌ Copying the WAR to the Wrong Directory

Deploy WAR files to:

```text
/opt/apache-tomcat-9.0.118/webapps
```

---

# 32. Troubleshooting

## Browser Shows 403 Forbidden

Possible causes:

* `RemoteAddrValve` is still enabled.
* Tomcat was not restarted after editing `context.xml`.

---

## Browser Shows 401 Unauthorized

Possible causes:

* Incorrect username or password.
* Missing `manager-gui` or `admin-gui` role.

---

## Application Returns 404

Possible causes:

* Incorrect URL.
* WAR deployment failed.
* Deployment is still in progress.
* Application context path is different.

---

## WAR File Not Extracted

Possible causes:

* Invalid WAR file.
* Insufficient permissions.
* Deployment error.

Check:

```bash
tail -f /opt/apache-tomcat-9.0.118/logs/catalina.out
```

---

## Tomcat Does Not Start

Check:

* Java installation.
* `JAVA_HOME` (if applicable).
* Execute permissions on the shell scripts.
* Configuration files for syntax errors.

---

# ✅ Part 2 Summary

You have now learned how to:

* Understand Tomcat configuration files
* Start, stop, and restart Tomcat
* Create global startup and shutdown commands
* Enable the Manager Application
* Configure Tomcat users and roles
* Enable the Host Manager
* Deploy a WAR file manually
* Verify deployments
* View and interpret Tomcat logs
* Troubleshoot common administration issues

---

# ▶️ Next Part

**Part 3** will cover the complete **Jenkins → Tomcat integration**, including:

* Installing the **Deploy to Container** plugin
* Configuring Tomcat credentials in Jenkins
* Setting up a Freestyle job for automatic WAR deployment
* Using Tomcat Manager for remote deployments
* End-to-end CI/CD flow
* Common Jenkins deployment errors
* Best practices for automated deployments

_____________________________________________________________________________________________________________________________________________________
# 🚀 Apache Tomcat 9.0.118 Installation & Jenkins Integration Guide

# Part 3A – Jenkins Integration Prerequisites & Deploy to Container Plugin

> **Prerequisites**
>
> Complete:
>
> * ✅ Part 1 – Tomcat Installation
> * ✅ Part 2 – Manager & Host Manager Configuration
> * ✅ Jenkins Installation
> * ✅ Maven Installation

---

# 📖 Table of Contents

* [33. CI/CD Architecture](#33-cicd-architecture)
* [34. How Jenkins Deploys to Tomcat](#34-how-jenkins-deploys-to-tomcat)
* [35. Before You Continue – Checklist](#35-before-you-continue--checklist)
* [36. Understanding Tomcat Roles](#36-understanding-tomcat-roles)
* [37. Configure `tomcat-users.xml`](#37-configure-tomcat-usersxml)
* [38. Install Deploy to Container Plugin](#38-install-deploy-to-container-plugin)
* [39. Configure Jenkins Credentials](#39-configure-jenkins-credentials)
* [40. Verification Checklist](#40-verification-checklist)
* [41. Next Part](#41-next-part)

---

# 33. CI/CD Architecture

Before configuring Jenkins, understand how the deployment works.

```text
                     Developer
                         │
                     git push
                         │
                         ▼
                GitHub Repository
                         │
                         ▼
                    Jenkins Job
                         │
                 Git Checkout
                         │
                         ▼
               Maven Clean Package
                         │
                  Generates WAR
                         │
                         ▼
          Deploy to Container Plugin
                         │
                         ▼
            Tomcat Manager Text API
                         │
                         ▼
                Apache Tomcat
                         │
                         ▼
                     Browser
```

---

> [!IMPORTANT]
>
> Jenkins **does not copy the WAR file directly** into Tomcat.
>
> Jenkins communicates with the **Tomcat Manager Application** using the **Manager Text API**.

---

# 34. How Jenkins Deploys to Tomcat

Many beginners think Jenkins simply copies the WAR file into:

```text
/opt/apache-tomcat/webapps
```

That is **not** what happens when using the **Deploy to Container Plugin**.

Instead, Jenkins performs the following steps:

```text
GitHub

↓

Clone Repository

↓

Maven Build

↓

target/application.war

↓

Deploy Plugin

↓

Tomcat Manager

↓

Tomcat deploys WAR

↓

Application Available
```

---

## Manager Text API

The Deploy Plugin communicates with Tomcat using URLs like:

```text
http://PUBLIC-IP:8080/manager/text/deploy
```

Other API endpoints include:

```text
/manager/text/list

/manager/text/reload

/manager/text/start

/manager/text/stop

/manager/text/undeploy
```

---

> [!NOTE]
>
> These endpoints are intended for scripts and automation, not for interactive browser use.

---

# 35. Before You Continue – Checklist

Verify each of the following.

| Requirement             | Status |
| ----------------------- | ------ |
| Java Installed          | ✅      |
| Maven Installed         | ✅      |
| Jenkins Running         | ✅      |
| Tomcat Running          | ✅      |
| Manager App Working     | ✅      |
| Port 8080 Open          | ✅      |
| GitHub Repository Ready | ✅      |

---

## Verify Tomcat

```bash
ps -ef | grep tomcat
```

---

## Verify Jenkins

Open

```text
http://JENKINS-IP:8080
```

---

## Verify Manager

Open

```text
http://TOMCAT-IP:8080/manager
```

Login successfully.

---

# 36. Understanding Tomcat Roles

This is one of the most misunderstood topics.

Tomcat provides several predefined roles.

| Role           | Purpose                 | Used By     |
| -------------- | ----------------------- | ----------- |
| manager-gui    | Access Manager Web UI   | Browser     |
| manager-script | Remote Deployment API   | Jenkins     |
| manager-status | Read-only Server Status | Monitoring  |
| manager-jmx    | JMX Management          | JMX Clients |
| admin-gui      | Host Manager            | Browser     |

---

## manager-gui

Allows access to

```text
http://IP:8080/manager
```

You can

* Deploy WAR manually
* Stop application
* Reload application

through the browser.

---

## manager-script

This is the **MOST IMPORTANT** role for Jenkins.

Without it,

the Deploy Plugin cannot deploy WAR files.

---

## manager-status

Provides read-only information.

Useful for

* Monitoring
* Dashboards

---

## manager-jmx

Provides JMX Proxy access.

Rarely used during DevOps training.

Mostly used in enterprise monitoring.

---

## admin-gui

Allows access to

```text
http://IP:8080/host-manager
```

---

# Why manager-gui is NOT Enough

This is where many people get confused.

You may successfully login to

```text
http://IP:8080/manager
```

using

```text
manager-gui
```

but Jenkins still fails.

Why?

Because Jenkins is NOT using the browser.

It is calling

```text
/manager/text/*
```

API endpoints.

Those require

```text
manager-script
```

permission.

---

Example

Browser

```text
manager-gui
```

✔ Works

---

Jenkins

```text
manager-script
```

✔ Required

---

Without it,

Jenkins usually reports

```text
FAIL - Unauthorized
```

or

```text
403 Forbidden
```

---

# 37. Configure tomcat-users.xml

Open

```bash
cd /opt/apache-tomcat-9.0.118/conf

vi tomcat-users.xml
```

Before

```xml
</tomcat-users>
```

add

```xml
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<role rolename="manager-status"/>
<role rolename="manager-jmx"/>
<role rolename="admin-gui"/>

<user
    username="ambika"
    password="ambika"
    roles="manager-gui,
           manager-script,
           manager-status,
           manager-jmx,
           admin-gui"/>
```

Save

Exit

Restart Tomcat

```bash
cd /opt/apache-tomcat-9.0.118/bin

./shutdown.sh

sleep 5

./startup.sh
```

---

## Verify

Open

```text
http://PUBLIC-IP:8080/manager
```

Login.

If login succeeds,

your configuration is correct.

---

> [!TIP]
>
> For a lab environment, using a single user with all required roles is convenient.
>
> In production, follow the principle of least privilege. For example, a Jenkins service account may only need `manager-script`, while administrators can have `manager-gui` and `admin-gui`.

---

# 38. Install Deploy to Container Plugin

Open Jenkins.

Navigate to

```text
Dashboard

↓

Manage Jenkins

↓

Plugins

↓

Available Plugins
```

Search

```text
Deploy to Container
```

Install

```text
Deploy to Container Plugin
```

Restart Jenkins if required.

---

## Verify

Open

```text
Manage Jenkins

↓

Plugins

↓

Installed Plugins
```

You should see

```text
Deploy to Container
```

---

# 39. Configure Jenkins Credentials

Open

```text
Dashboard

↓

Manage Jenkins

↓

Credentials

↓

System

↓

Global Credentials

↓

Add Credentials
```

Choose

```text
Username with Password
```

Fill

| Field       | Value                 |
| ----------- | --------------------- |
| Username    | ambika                |
| Password    | ambika                |
| ID          | tomcat                |
| Description | Apache Tomcat Manager |

Click

```text
Create
```

---

## Why Use Jenkins Credentials?

Never hardcode usernames or passwords inside Jenkins jobs.

Instead,

store them securely in the Jenkins Credentials Store.

Benefits:

* Secure
* Reusable
* Easy to update
* Avoids exposing passwords in job configurations

---

# 40. ✅ Verification Checklist

Before moving to **Part 3B**, verify the following:

| Task                                 | Completed |
| ------------------------------------ | --------- |
| Tomcat Running                       | ✅         |
| Manager App Accessible               | ✅         |
| `manager-script` Role Added          | ✅         |
| `admin-gui` Role Added               | ✅         |
| `manager-gui` Role Added             | ✅         |
| `tomcat-users.xml` Updated           | ✅         |
| Tomcat Restarted                     | ✅         |
| Deploy to Container Plugin Installed | ✅         |
| Jenkins Credentials Created          | ✅         |

---

# 41. 🎯 Next Part

In **Part 3B**, you will configure the Jenkins Freestyle project:

* Connect GitHub
* Configure Maven build
* Generate the WAR file
* Configure the **Deploy to Container** plugin
* Explain every field in the deployment configuration
* Deploy automatically to Tomcat
* Verify the deployment in both Tomcat Manager and the browser

_____________________________________________________________________________________________________________________________________________________
# 🚀 Apache Tomcat 9.0.118 Installation & Jenkins Integration Guide

# Part 3B – Jenkins Freestyle Job Configuration & Automatic WAR Deployment

> **Prerequisites**
>
> Complete:
>
> * ✅ Part 1 – Tomcat Installation
> * ✅ Part 2 – Manager App Configuration
> * ✅ Part 3A – Jenkins Prerequisites & Deploy to Container Plugin

---

# 📖 Table of Contents

* [40. Deployment Workflow](#40-deployment-workflow)
* [41. Before Creating the Jenkins Job](#41-before-creating-the-jenkins-job)
* [42. Create a Freestyle Project](#42-create-a-freestyle-project)
* [43. Configure Git Repository](#43-configure-git-repository)
* [44. Configure Build Triggers (Optional)](#44-configure-build-triggers-optional)
* [45. Configure Build Environment (Optional)](#45-configure-build-environment-optional)
* [46. Configure Maven Build](#46-configure-maven-build)
* [47. Configure Deploy to Container Plugin](#47-configure-deploy-to-container-plugin)
* [48. Build the Job](#48-build-the-job)
* [49. Verify Deployment](#49-verify-deployment)
* [50. Understanding Every Deployment Field](#50-understanding-every-deployment-field)
* [51. Common Deployment Mistakes](#51-common-deployment-mistakes)
* [52. Part 3B Summary](#52-part-3b-summary)
* [53. Next Part](#53-next-part)

---

# 40. Deployment Workflow

Before configuring Jenkins, understand what happens after clicking **Build Now**.

```text
                 Build Now
                     │
                     ▼
              Clone GitHub Code
                     │
                     ▼
            Download Dependencies
                     │
                     ▼
             Compile Java Source
                     │
                     ▼
              Run Unit Tests
                     │
                     ▼
                Package WAR File
                     │
                     ▼
         Deploy to Container Plugin
                     │
                     ▼
          Tomcat Manager Text API
                     │
                     ▼
             Deploy WAR to Tomcat
                     │
                     ▼
             Open Application
```

---

# 41. Before Creating the Jenkins Job

Verify the following:

| Requirement            | Status |
| ---------------------- | ------ |
| Jenkins Running        | ✅      |
| Git Installed          | ✅      |
| Maven Configured       | ✅      |
| JDK Configured         | ✅      |
| Tomcat Running         | ✅      |
| Manager App Accessible | ✅      |
| Credentials Created    | ✅      |

---

## Verify Jenkins Tools

Go to:

```text
Dashboard

↓

Manage Jenkins

↓

Tools
```

Verify:

| Tool  | Example   |
| ----- | --------- |
| JDK   | Java 21   |
| Maven | Maven 3.x |
| Git   | Installed |

---

> [!TIP]
>
> If Maven or JDK is not configured in **Manage Jenkins → Tools**, the build may fail even if they are installed on the server.

---

# 42. Create a Freestyle Project

Open Jenkins Dashboard.

Click

```text
New Item
```

Enter project name.

Example

```text
Java-WebApp-Deploy
```

Choose

```text
Freestyle Project
```

Click

```text
OK
```

---

# 43. Configure Git Repository

Scroll to

```text
Source Code Management
```

Select

```text
Git
```

Repository URL

```text
https://github.com/your-username/maven-web-application.git
```

Example

```text
https://github.com/ambika-k-devops/maven-web-application.git
```

Credentials

Select your GitHub credentials.

Branch

```text
*/main
```

or

```text
*/master
```

---

## Verification

Click

```text
Save
```

Then

```text
Build Now
```

Open

```text
Console Output
```

Expected

```text
Cloning the remote Git repository

Checking out Revision

Finished: SUCCESS
```

---

# 44. Configure Build Triggers (Optional)

This section is optional.

If you want Jenkins to build automatically:

Select one of the following.

### Poll SCM

```text
H/2 * * * *
```

Checks Git every 2 minutes.

---

### GitHub Webhook

Recommended for real projects.

GitHub

↓

Settings

↓

Webhooks

↓

Add Webhook

Payload

```text
http://JENKINS-IP:8080/github-webhook/
```

---

> [!NOTE]
>
> For learning purposes, **Build Now** is sufficient. You can configure webhooks later.

---

# 45. Configure Build Environment (Optional)

Depending on your project, you may enable options such as:

* Delete workspace before build starts
* Use secret text(s) or file(s)
* Inject environment variables

For a basic Java web application, you can leave these settings at their defaults.

---

# 46. Configure Maven Build

Scroll to

```text
Build
```

Click

```text
Add Build Step
```

Select

```text
Invoke top-level Maven targets
```

---

## Goals

Enter:

```text
clean package
```

---

## What Does `clean package` Do?

```text
clean
```

Deletes the previous build output.

---

```text
package
```

Compiles the project and creates the WAR file.

---

Flow

```text
Source Code

↓

Compile

↓

Run Tests

↓

Package

↓

WAR File
```

---

Expected Output

```text
target/

maven-web-application.war
```

---

## Verify

Console Output

```text
BUILD SUCCESS
```

---

# 47. Configure Deploy to Container Plugin

Scroll to

```text
Post-build Actions
```

Click

```text
Add Post-build Action
```

Choose

```text
Deploy war/ear to a container
```

---

The configuration screen contains several fields.

We'll explain each one.

---

## WAR/EAR Files

Enter

```text
**/*.war
```

---

### Why?

The plugin searches the Jenkins workspace for any WAR file.

Example

```text
workspace/

target/

maven-web-application.war
```

The pattern

```text
**/*.war
```

matches:

```text
target/maven-web-application.war
```

---

## Context Path

Leave it **blank**.

Tomcat will automatically use the WAR filename.

Example

WAR

```text
employee-service.war
```

Application URL

```text
http://IP:8080/employee-service
```

---

## Container

Choose

```text
Tomcat 9.x Remote
```

---

### Why "Remote"?

The plugin deploys through the Tomcat Manager API over HTTP.

It does **not** copy files directly to the server.

---

## Credentials

Select

```text
tomcat
```

(the credential created in Part 3A)

---

## Tomcat URL

Example

```text
http://43.xxx.xxx.xxx:8080
```

Do **not** use

```text
http://43.xxx.xxx.xxx
```

Port **8080** must be included.

---

> [!IMPORTANT]
>
> The URL should point to the Tomcat server, not the Jenkins server.

---

# 48. Build the Job

Click

```text
Save
```

Then click

```text
Build Now
```

Jenkins performs the following:

```text
Clone Repository

↓

Download Dependencies

↓

Compile Source

↓

Package WAR

↓

Deploy Plugin

↓

Tomcat Manager API

↓

Tomcat Deployment

↓

Application Running
```

---

## Console Output

Typical successful output:

```text
Building in workspace...

Compiling...

Packaging...

BUILD SUCCESS

Deploying WAR...

Deployment Successful

Finished: SUCCESS
```

---

# 49. Verify Deployment

## Verify in Tomcat Manager

Open

```text
http://PUBLIC-IP:8080/manager
```

You should see:

```text
Running

maven-web-application
```

---

## Verify in Browser

Open

```text
http://PUBLIC-IP:8080/maven-web-application
```

The application should load successfully.

---

## Verify Using Logs

```bash
tail -f /opt/apache-tomcat-9.0.118/logs/catalina.out
```

Look for messages indicating the application was deployed successfully.

---

# 50. Understanding Every Deployment Field

| Field         | Description                                |
| ------------- | ------------------------------------------ |
| WAR/EAR Files | Path to the WAR generated by Maven         |
| Context Path  | URL path for the deployed application      |
| Container     | Target server type (Tomcat 9.x Remote)     |
| Credentials   | Jenkins credentials for the Tomcat Manager |
| Tomcat URL    | Base URL of the Tomcat server              |

---

## Visual Summary

```text
GitHub Repository
        │
        ▼
Jenkins Job
        │
        ▼
Maven Build
        │
        ▼
target/*.war
        │
        ▼
Deploy Plugin
        │
        ▼
Tomcat Manager API
        │
        ▼
Apache Tomcat
        │
        ▼
Browser
```

---

# 51. Common Deployment Mistakes

## ❌ Using Only `manager-gui`

**Problem**

Browser login works.

Jenkins deployment fails.

**Reason**

`manager-script` role is missing.

**Solution**

Add:

```xml
<role rolename="manager-script"/>
```

Assign the role to the Jenkins user and restart Tomcat.

---

## ❌ Wrong WAR Pattern

Wrong:

```text
*.war
```

Correct:

```text
**/*.war
```

The recursive pattern is more reliable because it searches all subdirectories (such as `target/`).

---

## ❌ Incorrect Tomcat URL

Wrong:

```text
http://IP
```

Correct:

```text
http://IP:8080
```

---

## ❌ Wrong Credentials

Ensure the username and password in Jenkins match the values configured in `tomcat-users.xml`.

---

## ❌ Manager App Not Accessible

Verify:

* `context.xml` has been updated (lab setup).
* Tomcat has been restarted.
* Port 8080 is open.

---

## ❌ Maven Build Failed

Run the build manually:

```bash
mvn clean package
```

Resolve any compilation or dependency issues before attempting deployment through Jenkins.

---

# ✅ Part 3B Summary

You have now learned how to:

* Create a Jenkins Freestyle project
* Configure Git source code
* Configure Maven build steps
* Generate a WAR file
* Configure the Deploy to Container plugin
* Understand each deployment field
* Build and deploy the application automatically
* Verify deployment in Tomcat Manager, the browser, and Tomcat logs
* Identify and resolve common deployment mistakes

---

# ▶️ Next Part

**Part 3C** will complete the guide with:

* Advanced troubleshooting
* Best practices
* Production recommendations
* Tomcat security considerations
* Jenkins deployment scenarios
* Interview questions
* Complete command cheat sheet
* Final end-to-end CI/CD recap

_____________________________________________________________________________________________________________________________________________________
# 🚀 Apache Tomcat 9.0.118 Installation & Jenkins Integration Guide

# Part 3C – Troubleshooting, Best Practices, Interview Questions & Cheat Sheet

> **Prerequisites**
>
> Complete:
>
> * ✅ Part 1 – Tomcat Installation
> * ✅ Part 2 – Tomcat Administration
> * ✅ Part 3A – Jenkins Integration Prerequisites
> * ✅ Part 3B – Freestyle Job & Automatic Deployment

---

# 📖 Table of Contents

* [52. End-to-End CI/CD Workflow](#52-end-to-end-cicd-workflow)
* [53. Common Deployment Errors](#53-common-deployment-errors)
* [54. Troubleshooting Guide](#54-troubleshooting-guide)
* [55. Best Practices](#55-best-practices)
* [56. Production Recommendations](#56-production-recommendations)
* [57. Tomcat Directory Quick Reference](#57-tomcat-directory-quick-reference)
* [58. Frequently Used Commands](#58-frequently-used-commands)
* [59. Interview Questions](#59-interview-questions)
* [60. Complete Workflow Summary](#60-complete-workflow-summary)
* 
* [Congratulations!](#congratulations)

---

# 52. End-to-End CI/CD Workflow

Now let's understand the **complete deployment flow** from developer to browser.

```text
                    Developer
                        │
                  git add .
                        │
                  git commit
                        │
                   git push
                        │
                        ▼
               GitHub Repository
                        │
            Webhook / Poll SCM
                        │
                        ▼
                 Jenkins Server
                        │
                 Clone Repository
                        │
                        ▼
             Maven Clean Package
                        │
             Generates WAR File
                        │
                        ▼
      Deploy to Container Plugin
                        │
                        ▼
      Tomcat Manager Text API
                        │
                        ▼
               Apache Tomcat
                        │
                        ▼
                  End Users
```

---

## What Happens Internally?

```text
Developer pushes code

↓

GitHub stores source code

↓

Jenkins downloads code

↓

Maven compiles project

↓

WAR file created

↓

Deploy Plugin uploads WAR

↓

Tomcat extracts WAR

↓

Application starts

↓

Users access application
```

---

# 53. Common Deployment Errors

---

## Error 1

```text
FAIL - Unauthorized
```

### Cause

Wrong username/password

OR

`manager-script` role missing

### Solution

Check

```xml
tomcat-users.xml
```

Restart Tomcat

---

## Error 2

```text
403 Forbidden
```

### Cause

Manager App still restricted

### Solution

Comment

```xml
RemoteAddrValve
```

inside

```text
manager/META-INF/context.xml
```

Restart Tomcat

---

## Error 3

```text
404 Not Found
```

### Cause

Wrong Context Path

Open

```text
Manager App
```

Verify application name

---

## Error 4

```text
Connection Refused
```

### Cause

Tomcat stopped

Run

```bash
ps -ef | grep tomcat
```

---

## Error 5

```text
Address already in use
```

### Cause

Another application is using

```text
8080
```

Find process

```bash
lsof -i :8080
```

---

## Error 6

```text
BUILD FAILURE
```

### Cause

Maven Compilation Error

Run

```bash
mvn clean package
```

outside Jenkins

---

## Error 7

```text
Deployment Failed
```

### Cause

Wrong URL

Wrong Credentials

Manager App disabled

Wrong WAR

---

## Error 8

```text
Manager Application Login Failed
```

### Cause

Wrong password

Wrong role

Wrong XML syntax

---

## Error 9

```text
Application Deployed But Browser Shows Error
```

Check

```bash
tail -f catalina.out
```

---

## Error 10

```text
Jenkins says SUCCESS

Application Not Updated
```

Possible Causes

* Browser cache
* Old Context Path
* WAR not replaced
* Deployment failed silently

---

# 54. Troubleshooting Guide

## Problem

Tomcat won't start

### Check

```bash
java -version
```

---

Check

```bash
javac -version
```

---

Check

```bash
tail -100 catalina.out
```

---

## Problem

Tomcat starts then immediately stops

Check

```bash
tail -f catalina.out
```

Usually caused by

* Invalid XML
* Java issue
* Port conflict

---

## Problem

Cannot Login Manager App

Verify

```xml
manager-gui
```

exists

Restart Tomcat

---

## Problem

Jenkins Cannot Deploy

Verify

```xml
manager-script
```

exists

---

## Problem

Host Manager Not Opening

Verify

```text
host-manager/META-INF/context.xml
```

---

## Problem

Browser Cannot Open

Check

AWS Security Group

↓

Port

```text
8080
```

Open

---

Check Firewall

```bash
firewall-cmd --list-all
```

---

## Problem

WAR Not Extracted

Go

```bash
cd /opt/apache-tomcat/webapps
```

Check

```bash
ls
```

If WAR exists

But folder not created

Deployment failed

---

## Problem

Tomcat Not Listening

Run

```bash
lsof -i :8080
```

---

# 55. Best Practices

## 1.

Always use

```text
.tar.gz
```

instead of

```text
.zip
```

on Linux.

---

## 2.

Keep Tomcat under

```text
/opt
```

---

## 3.

Always restart after editing

* server.xml
* context.xml
* tomcat-users.xml

---

## 4.

Monitor

```bash
tail -f catalina.out
```

while deploying.

---

## 5.

Use meaningful WAR names.

Good

```text
employee-service.war

inventory-service.war
```

Bad

```text
test.war

project.war
```

---

## 6.

Keep Jenkins and Tomcat on separate servers for production.

---

## 7.

Backup

```text
conf/
```

before modifications.

---

## 8.

Never expose

```text
Manager App
```

to the Internet.

---

## 9.

Use

```text
manager-script
```

only for automation accounts.

---

## 10.

Keep Java and Tomcat versions compatible.

---

# 56. Production Recommendations

## Learning Environment

```text
Tomcat

↓

Manager App

↓

Public IP

↓

Jenkins
```

Perfectly fine.

---

## Production

```text
Internet

↓

Load Balancer

↓

Reverse Proxy

↓

Tomcat

↓

Private Network
```

Manager App

↓

Internal Network Only

---

Credentials

↓

Strong Password

---

Deployments

↓

Pipeline

---

Logs

↓

Centralized Logging

---

Backups

↓

Automated

---

# 57. Tomcat Directory Quick Reference

| Directory | Purpose            |
| --------- | ------------------ |
| bin       | Startup Scripts    |
| conf      | Configuration      |
| logs      | Log Files          |
| lib       | Libraries          |
| temp      | Temporary Files    |
| webapps   | Applications       |
| work      | Compiled JSP Files |

---

## Configuration Files

| File             | Purpose                   |
| ---------------- | ------------------------- |
| server.xml       | Server Configuration      |
| context.xml      | Application Configuration |
| tomcat-users.xml | Users & Roles             |
| web.xml          | Deployment Descriptor     |

---

# 58. Frequently Used Commands

## Start

```bash
cd /opt/apache-tomcat-9.0.118/bin

./startup.sh
```

---

## Stop

```bash
./shutdown.sh
```

---

## Restart

```bash
./shutdown.sh

sleep 5

./startup.sh
```

---

## Logs

```bash
tail -f /opt/apache-tomcat-9.0.118/logs/catalina.out
```

---

## Check Process

```bash
ps -ef | grep tomcat
```

---

## Check Port

```bash
lsof -i :8080
```

---

## Manager

```text
http://IP:8080/manager
```

---

## Host Manager

```text
http://IP:8080/host-manager
```

---

# 59. Interview Questions

## Basic

### What is Apache Tomcat?

---

### What is a WAR file?

---

### Difference between WAR and JAR?

---

### Why Tomcat?

---

### Why Tomcat instead of Apache HTTP Server?

---

### What is Catalina?

---

### What is Coyote?

---

### What is Jasper?

---

### What is the default Tomcat port?

---

### What is server.xml?

---

### What is context.xml?

---

### What is tomcat-users.xml?

---

### Difference between

```text
startup.sh
```

and

```text
catalina.sh
```

---

### What is Manager App?

---

### What is Host Manager?

---

### Why use manager-script?

---

### Why use manager-gui?

---

### What is Deploy to Container Plugin?

---

### How does Jenkins deploy to Tomcat?

---

### Explain the deployment flow.

Expected Answer

```text
Developer

↓

GitHub

↓

Jenkins

↓

Maven

↓

WAR

↓

Deploy Plugin

↓

Manager API

↓

Tomcat

↓

Browser
```

---

### Scenario

**Q**

Deployment successful

Browser shows

```text
404
```

How do you troubleshoot?

Answer

* Verify Context Path
* Check Manager App
* Check catalina.out
* Verify WAR deployed
* Verify webapps directory
* Restart Tomcat if required

---

### Scenario

**Q**

Browser login works

Jenkins deployment fails

Why?

Answer

Browser uses

```text
manager-gui
```

Jenkins uses

```text
manager-script
```

---

# 60. Complete Workflow Summary

## Installation

```text
Install Java

↓

Install Packages

↓

Download Tomcat

↓

Extract

↓

Give Permissions

↓

Start Tomcat

↓

Verify
```

---

## Configuration

```text
Manager App

↓

context.xml

↓

tomcat-users.xml

↓

Restart

↓

Verify Login
```

---

## Deployment

```text
GitHub

↓

Jenkins

↓

Maven

↓

WAR

↓

Deploy Plugin

↓

Manager API

↓

Tomcat

↓

Browser
```

---

# 🎉 Congratulations!

You have completed a full **end-to-end Apache Tomcat and Jenkins integration**.

By following Parts **1, 2, 3A, 3B, and 3C**, you now know how to:

* Install Apache Tomcat 9.0.118
* Configure Java 21
* Manage Tomcat services
* Enable Manager and Host Manager
* Configure Tomcat users and roles
* Understand the difference between `manager-gui` and `manager-script`
* Deploy WAR files manually
* Integrate Jenkins with Tomcat
* Configure the Deploy to Container plugin
* Automate deployments
* Troubleshoot common deployment issues
* Explain the entire CI/CD deployment flow in interviews

> **Final Tip**
>
> Keep this README updated whenever you encounter a new issue or learn a better practice. Over time, it becomes a living document that reflects your real-world experience rather than a static set of notes.

_____________________________________________________________________________________________________________________________________________________

# AWS instances for Jenkins, Tomcat, SonarQube, Nexus
<img width="1361" height="674" alt="Screenshot 2026-06-27 015815" src="https://github.com/user-attachments/assets/21c0e6cb-a693-4099-8f40-3a85eca4787b" />

# tomcat
<img width="1363" height="719" alt="Screenshot 2026-06-26 234921" src="https://github.com/user-attachments/assets/514da4cb-284f-48e5-beae-a41e27f746d6" />

# before setup
<img width="1364" height="709" alt="Screenshot 2026-06-26 234932" src="https://github.com/user-attachments/assets/e84fd4f9-dc98-4c00-acc4-a0af564d11cd" />

# tomcat setup in jenkins
<img width="1365" height="767" alt="Screenshot 2026-06-27 012131" src="https://github.com/user-attachments/assets/98f215ee-5714-4431-9b4c-602510774998" />

# build success in all stages 
    ## Github + Maven + Jenkins + SOnarQube + Nexus + Tomcat
<img width="1363" height="767" alt="Screenshot 2026-06-27 012317" src="https://github.com/user-attachments/assets/3b9227c9-f1f1-468d-bca0-f32ebb587861" />

# SonarQube
<img width="1366" height="768" alt="Screenshot 2026-06-27 012342" src="https://github.com/user-attachments/assets/f1a62565-8cc8-4aa0-8dc3-950c888dcc10" />

# Nexus
<img width="1366" height="768" alt="Screenshot 2026-06-27 012508" src="https://github.com/user-attachments/assets/e9ec5999-4a71-4701-ba12-77a453841bb1" />

# Tomcat
<img width="1366" height="768" alt="Screenshot 2026-06-27 012706" src="https://github.com/user-attachments/assets/e07045d0-7707-48e1-8e74-e8ff81cc03c6" />

# tomcat after setup
<img width="1360" height="727" alt="Screenshot 2026-06-27 005350" src="https://github.com/user-attachments/assets/e90c724e-509b-43e1-a7d6-ac7ed01c1ade" />

------------------------ Jenkins few other screens while practice related to jenkins plugins (free style job)
# MobaXterm : setting.xml
<img width="1365" height="754" alt="Screenshot 2026-06-27 004418" src="https://github.com/user-attachments/assets/6635e96a-5861-4588-be50-a36b44b8409f" />
# 1
<img width="1366" height="768" alt="Screenshot 2026-06-27 013109" src="https://github.com/user-attachments/assets/07bdbe6d-474e-4716-b3bd-7e739989001c" />
# 2
<img width="1366" height="768" alt="Screenshot 2026-06-27 013451" src="https://github.com/user-attachments/assets/f38d79b5-f135-4f81-9930-1bfab2795f01" />
# 3
<img width="1366" height="768" alt="Screenshot 2026-06-27 013530" src="https://github.com/user-attachments/assets/6ec6e5fe-ffab-4af3-8694-71980f18a76a" />
# 4 Blue Ocean Plugin in Jenkins
<img width="1366" height="768" alt="Screenshot 2026-06-27 014517" src="https://github.com/user-attachments/assets/dec2e5e5-033a-4169-9609-746993f1032b" />
# Build Monitor View Plugin
<img width="1366" height="768" alt="Screenshot 2026-06-27 015111" src="https://github.com/user-attachments/assets/512e3f92-8fdd-4c1b-98ec-c8ada01ab803" />
# All Builds View in Jenkins 
<img width="1366" height="768" alt="Screenshot 2026-06-27 015217" src="https://github.com/user-attachments/assets/70f1012a-5b25-438c-9e88-48345e7c2d04" />

--------------------------------------------------------------------------------------------------------------------------------------------------------

# 👨‍💻 Author

## Ambika Rama Katyayini Kachetti

**DevOps Engineer | AWS | Jenkins | Docker | Kubernetes | Terraform | Linux | CI/CD**

Passionate about building scalable DevOps solutions and automating software delivery pipelines. This repository documents my hands-on learning journey, covering DevOps tools, cloud technologies, and end-to-end CI/CD implementations through practical examples and real-world projects.

---

## 📌 Connect with Me

* **GitHub:** https://github.com/ambika-k-devops
* **LinkedIn:** www.linkedin.com/in/ambika-rama-katyayini-k
* **Email:** ambika9779@gmail.com

---

## 📚 About This Repository

This repository is a collection of my personal DevOps learning notes, hands-on labs, interview preparation material, and end-to-end implementation guides.

Topics covered include:

* Linux Administration
* Git & GitHub
* Apache Maven
* Jenkins
* SonarQube
* Nexus Repository
* Apache Tomcat
* Docker
* Kubernetes
* Terraform
* AWS Cloud
* CI/CD Pipeline Projects

---

## ⭐ Support

If you found this repository useful:

* ⭐ Star this repository
* 🍴 Fork it
* 🛠️ Try the hands-on examples
* 📖 Use it as a reference for DevOps interview preparation

---

## 📄 License

This project is licensed under the **MIT License**.

Feel free to use the content for learning and educational purposes.

---

<p align="center">

### 🚀 Happy Learning & Happy Automating! 🚀

**"Learn • Build • Automate • Deploy • Repeat"**

</p>




