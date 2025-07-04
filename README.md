
# Azure Load Balancer Assignment – Full Setup Guide with Explanation

This document provides a **complete step-by-step guide** to creating and verifying both **External (Public)** and **Internal (Private)** Load Balancers in Microsoft Azure using Linux VMs. It includes configuration steps, explanation of each component, and screenshot suggestions for reporting or GitHub submission.

---

## 🌐 Overview

We deployed a highly available architecture consisting of:
- Two Linux Virtual Machines (VM1 and VM2)
- An **External Load Balancer (ELB)** accessible via public internet
- An **Internal Load Balancer (ILB)** used for internal VM-to-VM traffic
- Proper Network Security Group (NSG) rules

```
External Client
     |
     v
[External Load Balancer (Public IP)]
     |
     v
[VM1] <--> [VM2]     <- also used by Internal Load Balancer
     ^
     |
[Internal Load Balancer (Private IP 10.0.1.100)]
```

---

## 📦 Resources Used

| Resource              | Details                                       |
|----------------------|-----------------------------------------------|
| Virtual Machines     | 2 Ubuntu Linux VMs with Apache installed       |
| Load Balancers       | 1 Public, 1 Internal                          |
| Virtual Network      | Custom VNet with Subnets (web-subnet)         |
| NSG (Firewall)       | Allow port 80 (HTTP) from Internet + VNet     |
| Health Probes        | HTTP-based, used by Load Balancers            |

---

## 🖥️ Step-by-Step Setup Instructions

### Step 1: Create Two Linux VMs (VM1 and VM2)
- Image: Ubuntu 20.04 LTS
- Size: B1s or Standard_B2ms
- In **same VNet and subnet** (e.g., `web-subnet`)
- Add inbound NSG rule to allow **HTTP (port 80)**

### Step 2: Install Apache on Both VMs
SSH into each VM and run:
```bash
sudo apt update
sudo apt install apache2 -y
```
Customize homepage for testing:
```bash
echo "Hello from VM1" | sudo tee /var/www/html/index.html   # For VM1
echo "Hello from VM2" | sudo tee /var/www/html/index.html   # For VM2
```

---

## 🌍 Create External Load Balancer

### Configuration:
1. **Create Load Balancer** → Type: `Public`
2. **Frontend IP**:
   - Create new Public IP (Static)
   - Name: `PublicFrontEnd`
3. **Backend Pool**:
   - Name: `ExternalBackendPool`
   - Add VM1 and VM2 by NIC
4. **Health Probe**:
   - Protocol: HTTP, Port: 80, Path: `/`
5. **Load Balancing Rule**:
   - Name: `http-rule`
   - Frontend Port: 80 → Backend Port: 80
   - Protocol: TCP
   - Associate Health Probe

### Verification:
- Get the **Public IP** from Azure Portal
- Open in browser:
```
http://<external-public-ip>
```
- Expected alternating response:
  - "Hello from VM1"
  - "Hello from VM2"

📸 **Screenshot Suggestions**:
- Apache welcome page: `apache-running.png`
- Browser with Public IP: `external-lb-test.png`
- Azure LB backend pool: `external-backend-pool.png`

---

## 🔒 Create Internal Load Balancer

### Configuration:
1. **Create Load Balancer** → Type: `Internal`
2. **Frontend IP Configuration**:
   - Virtual Network: `MyVNet`
   - Subnet: `web-subnet`
   - Assignment: Static IP → `10.0.1.100`
   - Name: `InternalFrontEnd`
3. **Backend Pool**:
   - Name: `InternalBackendPool`
   - Add VM1 and VM2
4. **Health Probe**:
   - HTTP, Port 80, Path: `/`
5. **Load Balancing Rule**:
   - Name: `internal-http-rule`
   - Frontend Port: 80 → Backend Port: 80
   - Associate probe & backend pool

### NSG Important Note:
- Ensure NSG rules **allow HTTP traffic from within Virtual Network**
  - Source: VirtualNetwork
  - Destination Port: 80
  - Protocol: TCP
  - Action: Allow

### Verification:
- SSH into VM1:
```bash
curl http://10.0.1.100
```
- Expected alternating output from VM1 or VM

---

## ✅ Test Result Summary

| Component                | Status     | Notes                           |
|-------------------------|------------|---------------------------------|
| Apache on VM1 & VM2     | ✅ Running | Responds to curl/browser        |
| External LB             | ✅ Working | Verified via browser            |
| Internal LB             | ✅ Working | Verified via internal curl      |
| NSG Inbound Rules       | ✅ Set     | Port 80 open from Any + VNet    |

---

## 📦 Suggested GitHub Repo Files

| File Name                  | Description                                 |
|---------------------------|---------------------------------------------|
| `README.md`               | Full documentation                         |
| `external-lb-test.png`    | Browser screenshot of external LB test     |
| `internal-lb-curl.png`    | Terminal curl test to internal LB IP       |
| `apache-running.png`      | Apache home page on VM                     |
| `backend-pool.png`        | Backend pool with VMs added                |
| `frontend-ip-config.png`  | Frontend config for both LBs               |
| `load-balancing-rule.png` | Load balancing rule for port 80            |

---

## 🏁 Final Thoughts
- You’ve successfully created a complete load-balanced architecture in Azure
- Verified both internet-facing and internal communication across VMs
- Proper NSG rules and IP configurations make it secure and functional

Feel free to build upon this setup with:
- Auto-scaling VM scale sets
- Application Gateway with HTTPS termination
- Integration with Azure Monitor or Application Insights
