# Day 08 - Cloud Server Setup: Nginx Deployment on AWS EC2 🚀

## Objective

The goal of today's task was to launch a cloud server, connect to it using SSH, deploy an Nginx web server, configure network access, and understand how to troubleshoot common deployment issues.

---

# Environment

* AWS EC2 (Ubuntu)
* Linux
* Nginx
* SSH
* AWS Security Groups

---

# Step 1 - Launch EC2 Instance

Created a new Ubuntu EC2 instance in AWS.

* Generated a new key pair (.pem)
* Downloaded the private key
* Launched the instance successfully

---

# Step 2 - Connect to EC2

Connected to the server using AWS EC2 Instance Connect.

Verified connection by checking the hostname and current user.

Commands used:

```bash
whoami
hostname
pwd
```

---

# Step 3 - Update the Server

Updated package information before installing any software.

```bash
sudo apt update
```

---

# Step 4 - Install Nginx

Installed the Nginx web server.

```bash
sudo apt install nginx -y
```

Verified installation:

```bash
nginx -v
```

Checked service status:

```bash
systemctl status nginx
```

The service was running successfully.

---

# Step 5 - Configure Security Group

Initially, the Nginx Welcome Page was not opening in the browser.

### Issue Faced

The browser showed:

```
ERR_CONNECTION_TIMED_OUT
```

### Root Cause

The EC2 Security Group allowed only SSH (Port 22).

HTTP traffic (Port 80) was not allowed.

### Solution

Added a new inbound rule:

* Type: HTTP
* Protocol: TCP
* Port: 80
* Source: 0.0.0.0/0

After adding the rule, the Nginx Welcome Page became accessible from the browser.

---

# Step 6 - Verify Nginx

Verified locally using:

```bash
curl localhost
```

Output displayed the default Nginx Welcome Page.

Also verified from the browser using the EC2 Public IP.

---

# Step 7 - View Nginx Logs

Viewed recent access logs.

```bash
sudo tail -20 /var/log/nginx/access.log
```

This showed incoming HTTP requests made to the server.

---

# Step 8 - Save Logs

Copied the access logs into a new file.

```bash
sudo cp /var/log/nginx/access.log ~/nginx-logs.txt
```

---

# Issue Faced While Saving Logs

When trying to open the file:

```bash
cat ~/nginx-logs.txt
```

I received:

```
Permission denied
```

### Root Cause

The file was owned by the root user because it was copied using `sudo`.

Verified ownership:

```bash
ls -l ~/nginx-logs.txt
```

Output:

```
-rw-r----- 1 root root ...
```

### Solution

Changed the ownership of the file.

```bash
sudo chown ubuntu:ubuntu ~/nginx-logs.txt
```

Verified again:

```bash
ls -l ~/nginx-logs.txt
```

Successfully viewed the logs.

```bash
cat ~/nginx-logs.txt
```

---

# Commands Used

```bash
sudo apt update
sudo apt install nginx -y
nginx -v
systemctl status nginx
curl localhost
sudo tail -20 /var/log/nginx/access.log
sudo cp /var/log/nginx/access.log ~/nginx-logs.txt
ls -l ~/nginx-logs.txt
sudo chown ubuntu:ubuntu ~/nginx-logs.txt
cat ~/nginx-logs.txt
```

---

# Challenges Faced

### Challenge 1

Nginx webpage was not opening in the browser.

**Reason**

HTTP Port 80 was blocked in the Security Group.

**Solution**

Added an inbound rule for HTTP (Port 80).

---

### Challenge 2

Unable to read `nginx-logs.txt`.

**Reason**

The file owner was `root`.

**Solution**

Changed ownership using:

```bash
sudo chown ubuntu:ubuntu ~/nginx-logs.txt
```

---

# What I Learned

* How to launch and access an AWS EC2 instance.
* How to install and verify an Nginx web server.
* How AWS Security Groups control inbound traffic.
* How to troubleshoot website accessibility issues.
* How to view and manage Nginx log files.
* Difference between file permissions and file ownership.
* How to use `chown` to resolve permission-related issues.

---

# Screenshots

* SSH Connection
* Nginx Installation
* Nginx Service Status
* AWS Security Group (HTTP Port 80)
* Nginx Welcome Page
* Nginx Access Logs

---

# Key Takeaway

Deploying a web server is more than just installing software. Understanding networking, security groups, Linux permissions, and log analysis is equally important. Today's hands-on practice helped me understand how to troubleshoot deployment issues step by step, just like in a real production environment.

---

## Day 08 Completed ✅

**#90DaysOfDevOps**
