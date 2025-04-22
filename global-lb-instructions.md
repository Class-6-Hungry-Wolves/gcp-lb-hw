# GCP Global Load Balancer Setup Guide

This guide walks you through the process of setting up a global HTTP load balancer in Google Cloud Platform (GCP) using the GUI. No command line experience is needed.

**Assume default settings for anything not mentioned.**

---

## 1. Create a Virtual Private Cloud (VPC)

1. Go to **VPC network > VPC networks**
2. Click **Create VPC network**
3. Set the following values:

- **Name:** `globalvpc01`
- **Subnets:**
  - **Name:** `netherlandssubnet01`
    - **Region:** `europe-west4`
    - **IP Range:** `10.80.36.0/24`
    - **Private Google Access:** On
  - **Name:** `osakasubnet01`
    - **Region:** `asia-northeast2`
    - **IP Range:** `10.80.46.0/24`
    - **Private Google Access:** On
    - **Flow Logs:** On

- **Routing Mode:** Global

4. Click **Create**
![alt text](image.png)
![alt text](image-1.png)

---

## 2. Create Cloud NAT for Internet Access

1. Go to **NAT gateways** under **Hybrid Connectivity > NAT**
2. Click **Create NAT gateway**

### Netherlands NAT:
- **Name:** `globalvpcnat01`
- **Network:** `globalvpc01`
- **Region:** `europe-west4`
- **Cloud Router:**
  - Click **Create new**
  - **Name:** `netherlandcloudrouter01`

Click **Create**

### Osaka NAT:
- **Name:** `globalvpcnat02`
- **Network:** `globalvpc01`
- **Region:** `asia-northeast2`
- **Cloud Router:**
  - Click **Create new**
  - **Name:** `osakacloudrouter01`

Click **Create**

![alt text](image-3.png)

---

## 3. Create Instance Templates

1. Go to **Compute Engine > Instance templates**
2. Click **Create instance template**

### Netherlands Template:
- **Name:** `netherlandinstancetemplate01`
- **Location:** Global
- **Networking Tags:** `globalvpc01-allow-http`, `globalvpc01-allow-healthcheck`
- **Network Interface:**
  - **Network:** `globalvpc01`
  - **Subnetwork Region:** `europe-west4`
  - **External IP:** None
- **Startup Script:**
  - Visit https://github.com/Class-6-Hungry-Wolves/GCP-startup-script-template/blob/main/new-class-template.sh
  - Click **Raw**
  - Copy the contents and paste it into the **Startup script** section

Click **Create**

### Osaka Template:
- **Name:** `osakainstancetemplate01`
- **Location:** Global
- **Networking Tags:** Same as above
- **Network Interface:**
  - **Network:** `globalvpc01`
  - **Subnetwork Region:** `asia-northeast2`
  - **External IP:** None
- **Startup Script:**
  - Visit https://github.com/Class-6-Hungry-Wolves/GCP-startup-script-template/blob/main/other-new-class-script-wolves.sh
  - Click **Raw**
  - Copy the contents and paste it into the **Startup script** section

Click **Create**


![alt text](image-4.png)
![alt text](image-5.png)
![alt text](image-6.png)
![alt text](image-7.png)
![alt text](image-8.png)

---

## 4. Create Instance Groups

1. Go to **Compute Engine > Instance groups**
2. Click **Create instance group**

### Netherlands Group:
- **Name:** `netherland-instance-group01`
- **Instance template:** `netherlandinstancetemplate01`
- **Location Type:** Multiple zones (in `europe-west4`)
- **Autoscaling:** Min = 3, Max = 6
- **Health Check:**
  - Click **Create health check**
  - **Name:** `globalvpc01hc`
  - **Logs:** On
  - **Interval:** 10 seconds
  - **Timeout:** 10 seconds
  - **Initial Delay:** 60 seconds

Click **Create**

### Osaka Group:
- **Name:** `osaka-instance-group01`
- **Instance template:** `osakainstancetemplate01`
- **Location Type:** Multiple zones (in `asia-northeast2`)
- **Autoscaling:** Min = 3, Max = 6
- **Health Check:** Same as above

Click **Create**

![alt text](image-9.png)
![alt text](image-10.png)
![alt text](image-11.png)
![alt text](image-12.png)
![alt text](image-13.png)
![alt text](image-14.png)
![alt text](image-15.png)
![alt text](image-16.png)

---

## 5. Create Firewall Rules

1. Go to **VPC network > Firewall rules**
2. Click **Create firewall rule** and fill out the following for each:

### Allow HTTP:
- **Name:** `globalvpc01-allow-http`
- **Target Tags:** `globalvpc01-allow-http`
- **Source IP Ranges:** `0.0.0.0/0`
- **Protocols/Ports:** TCP: 80
- **Logs:** On

### Allow Health Check:
- **Name:** `globalvpc01-allow-healthcheck`
- **Target Tags:** `globalvpc01-allow-healthcheck`
- **Source IP Ranges:** `0.0.0.0/0`
- **Protocols/Ports:** TCP: 80
- **Logs:** On

### Bastion Host - SSH:
- **Name:** `globalvpc01-bastion`
- **Target Tags:** `globalvpc01-allow-ssh`
- **Source IP Ranges:** `0.0.0.0/0`
- **Protocols/Ports:** TCP: 22
- **Logs:** On

### Bastion Host - RDP:
- **Name:** `globalvpc01-allow-rdp`
- **Target Tags:** `globalvpc01-bastion-host`
- **Source IP Ranges:** `0.0.0.0/0`
- **Protocols/Ports:** TCP: 3389
- **Logs:** On

![alt text](image-17.png)
![alt text](image-18.png)
![alt text](image-19.png)
![alt text](image-20.png)
![alt text](image-21.png)
![alt text](image-22.png)
![alt text](image-23.png)

---

## 6. Create Load Balancer

1. Go to **Network services > Load balancing**
2. Click **Create Load Balancer**

### Settings:
- **Type:** Application Load Balancer
- **Internet Facing:** Yes (External)
- **Global:** Yes

### Frontend Configuration:
- **Name:** `globalvpclb01`
- **Protocol:** HTTP

### Backend Configuration:
- **Backend Service Name:** `globalvpc01-backend01`
- **Backends:**
  - Add `netherland-instance-group01` (Port: 80)
  - Add `osaka-instance-group01` (Port: 80)
- **Enable Cloud CDN**
- **Health Check:**
  - Click **Create new health check**
  - **Name:** `globalvpc01backendhc01`
  - **Interval:** 10s
  - **Timeout:** 10s
- **Logging:** Enable Logging

If the policy name is too long to save, shorten it slightly (e.g., `vpc-backend01`).

Click **Create**

Wait a few minutes for provisioning.

Once finished, go to the Load Balancer and copy the **IP address**, then paste it into your browser to verify

![alt text](image-24.png)
![alt text](image-25.png)
![alt text](image-26.png)
![alt text](image-27.png)
![alt text](image-28.png)
![alt text](image-29.png)
![alt text](image-30.png)
![alt text](image-31.png)
![alt text](image-32.png)
![alt text](image-33.png)

---

## 7. Test Other Region Page

### Terminal Test (Bastion Host):

1. Create a VM:
   - **Name:** `osaka-bastion-host`
   - **Region:** `asia-northeast2`
   - **Network Tags:** `globalvpc01-bastion`, `globalvpc01-allow-http`
   - **External IP:** None

2. Click **Create**
3. SSH into the VM
4. Run the command below:

curl http://{your-loadbalancer-ip}


# GCP Visual Test: Verify Load Balancer with RDP Instance

This guide helps you visually confirm that your GCP Load Balancer is routing traffic correctly by creating a Windows Server RDP instance in a different region and accessing the Load Balancer IP from a browser.

---

## Step 1: Create an RDP Instance in Osaka Region

1. Go to **Compute Engine > VM instances**
2. Click **Create Instance**
3. Set the following configuration:

- **Name:** `osaka-rdp`
- **Region:** `asia-northeast2`
- **Zone:** (Pick any zone in this region)
- **Machine Type:** (Use e2-medium or similar)
- **Boot Disk:**
  - **Operating System:** Windows Server
  - **Version:** Windows Server 2022 Datacenter

- **Firewall:**
  - Allow RDP traffic (if not already done via firewall rule)

- **Networking:**
  - **Network:** `globalvpc01`
  - **Subnetwork:** `osakasubnet01`
  - **External IP:** None
  - **Network Tags:** `globalvpc01-bastion`, `globalvpc01-allow-http`

4. Click **Create**

---

## Step 2: Connect to the RDP Instance

1. Once the instance is created, click the instance name
2. Click **Set Windows Password** to generate a username and password
3. Download the **.rdp** file and open it with Remote Desktop
4. Use the generated credentials to log in

---

## Step 3: Access Load Balancer IP

1. Once inside the Windows desktop:
   - Open **Microsoft Edge**
   - Complete the initial Edge setup (if prompted)
2. In the address bar, type the Load Balancer IP: http://{your-loadbalancer-ip}
3. You should see the web page content specific to the Osaka instance group, confirming the load balancer is routing traffic correctly from that region.

![alt text](image-34.png)

