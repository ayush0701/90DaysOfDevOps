# Nginx Deployment on AWS EC2

This project documents launching a cloud instance, installing and configuring Nginx, opening it up for public web access, and verifying the deployed page is reachable from the internet.

## Task Overview

- [x] Launch a cloud instance (AWS EC2)
- [x] Connect via SSH
- [x] Install Nginx
- [x] Configure security group for web access (port 80)
- [x] Extract and save logs to a file
- [x] Verify the webpage is accessible from the internet

## 1. EC2 Instance

Instance launched on AWS EC2 (`t3.micro`, Ubuntu, `us-east-1d`), status checks passed (3/3), with security group `launch-wizard-1` attached.

![EC2 Instance Running](screenshots/01-ec2-instance-running.png)

- **Instance ID:** i-097d4cd14f433f8da
- **Public IPv4:** 18.208.231.96
- **Security Group:** sg-02c747559ba215c1e (launch-wizard-1)

## 2. SSH Connection & Nginx Install

Connected to the instance via SSH and installed Nginx:

```bash
ssh -i your-key.pem ubuntu@18.208.231.96

sudo apt update
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl status nginx
```

A custom portfolio page (`index.html`) was deployed to `/var/www/html/` to replace the default Nginx landing page.

## 3. Security Group Configuration

Inbound rule added to the security group to allow public HTTP traffic:

| Type | Protocol | Port | Source    |
|------|----------|------|-----------|
| HTTP | TCP      | 80   | 0.0.0.0/0 |
| SSH  | TCP      | 22   | My IP     |

## 4. Logs

Nginx service and access logs were extracted and saved to file on the server:

```bash
journalctl -u nginx > nginx_log.txt
sudo cp /var/log/nginx/access.log ~/nginx_access_log.txt
sudo cp /var/log/nginx/error.log ~/nginx_error_log.txt
```

Confirmed in the working directory on the instance:

![Terminal showing nginx_log.txt](screenshots/03-terminal-log-file.png)

The generated log file, [`nginx_log.txt`](nginx_log.txt), is included in this repo as proof of capture.

## 5. Live Verification

The deployed page is publicly accessible at:

**http://18.208.231.96:80**

![Portfolio page live in browser](screenshots/02-nginx-portfolio-live.png)

Verified externally with:

```bash
curl -I http://18.208.231.96:80
```

Expected response: `HTTP/1.1 200 OK`

## Repo Structure

```
.
├── README.md
├── nginx_log.txt
└── screenshots/
    ├── 01-ec2-instance-running.png
    ├── 02-nginx-portfolio-live.png
    └── 03-terminal-log-file.png
```
