# TCP/HTTP Connectivity Troubleshooting

## Practical 1 — Cloud Support Networking Lab

## 1. Project Overview

This practical demonstrates a structured approach to troubleshooting TCP and HTTP connectivity to an Nginx web server hosted on an AWS EC2 instance.

The objective was to determine why HTTP connectivity from a Windows client to an Ubuntu EC2 instance was initially failing, systematically eliminate potential network and host-level causes, identify the root cause, and restore connectivity.

The investigation followed a layered Cloud Support troubleshooting methodology:

**Client → Public IP → Internet Gateway → VPC → Network ACL → Security Group → EC2 → Host Firewall → Nginx → HTTP Response**

---

## 2. Objective

Troubleshoot an HTTP connectivity failure to an Nginx web server running on an AWS EC2 instance.

The investigation focused on:

- Verifying that the web service was running.
- Confirming that TCP port 80 was listening.
- Testing HTTP connectivity locally on the server.
- Verifying AWS Security Group rules.
- Checking the VPC route table.
- Checking Network ACL rules.
- Checking host-level firewall configuration.
- Testing connectivity from an external Windows client.
- Identifying the root cause.
- Restoring and validating HTTP connectivity.

---

## 3. Environment
| Component | Configuration |
|---|---|
| Cloud Provider | AWS |
| Compute | Amazon EC2 |
| Operating System | Ubuntu 24.04 LTS |
| Web Server | Nginx |
| Protocol | HTTP |
| HTTP Port | TCP 80 |
| SSH Port | TCP 22 |
| Client | Windows PowerShell |
| Network | AWS VPC |

The EC2 instance hosts a custom Nginx webpage created as part of my Cloud Support / Cloud Engineer hands-on lab portfolio.

---

## 4. Problem Statement

An external HTTP connectivity test from Windows PowerShell initially failed when connecting to the EC2 instance.

The initial test indicated that TCP port 80 was unreachable.

Rather than immediately modifying AWS networking or firewall configuration, the investigation followed a layered troubleshooting process to determine where connectivity was being lost.

---

## 5. Troubleshooting Methodology

The investigation followed an inside-out approach:

1. Verify the application.
2. Verify the listening TCP port.
3. Test the service locally.
4. Check host-level firewall controls.
5. Check AWS Security Group rules.
6. Check VPC routing.
7. Check Network ACL rules.
8. Verify the external destination.
9. Test connectivity from the client.
10. Validate the HTTP response.

This approach helped eliminate potential causes using evidence rather than making unnecessary configuration changes.
## 6. Investigation

### 6.1 Verify Listening Services

The EC2 instance was checked to confirm that the expected services were listening.

```bash
sudo ss -tulnp
The results confirmed:

- SSH was listening on TCP port 22.
- Nginx was listening on TCP port 80.

**Finding:** The operating system had a process listening on the expected HTTP port.
---

### 6.2 Test Nginx Locally

Nginx was tested directly from the EC2 instance.

```bash
curl http://localhost
---

### 6.3 Verify the EC2 Security Group

The EC2 Security Group was reviewed to determine whether inbound HTTP traffic was permitted.

The relevant rule allowed:

```text
Protocol: TCP
Port: 80
Source: 0.0.0.0/0
---

### 6.4 Check UFW

The Ubuntu host firewall was checked.

```bash
sudo ufw status
---

### 6.5 Verify VPC Routing

The VPC route table was reviewed to confirm that Internet-bound traffic had a valid route.

The relevant route was:

```text
Destination: 0.0.0.0/0
Target: Internet Gateway
---

### 6.6 Check Network ACLs

The Network ACL configuration was reviewed for inbound and outbound traffic.

The applicable rules allowed the required traffic.

**Finding:** The Network ACL was not blocking the HTTP connection.
---

### 6.7 Check iptables

The host's iptables configuration was inspected for packet-filtering rules.

```bash
sudo iptables -L -n -v
---

### 6.8 Check nftables

Because modern Ubuntu systems can use nftables, the nftables ruleset was also checked.

```bash
sudo nft list ruleset
---

### 6.9 Verify the EC2 Public IP

After the server-side and AWS networking checks were completed, the EC2 public IPv4 address was reviewed.

The EC2 instance had previously been stopped and restarted.

The investigation revealed that its public IPv4 address had changed.

The external connectivity test had been using the **old public IP address**.

This explained why:

- Nginx was working.
- TCP port 80 was listening.
- The Security Group allowed HTTP.
- UFW was inactive.
- The VPC route was correct.
- Network ACLs allowed traffic.
- iptables was not blocking traffic.
- nftables was not blocking traffic.
---

## 7. Root Cause

The root cause was an **outdated EC2 public IP address being used for the external connectivity test**.

The EC2 instance's public IPv4 address changed after the instance was stopped and restarted.

The client was therefore testing the wrong destination.

---

## 8. Resolution

The issue was resolved by identifying and using the EC2 instance's current public IPv4 address.

The current address is intentionally represented as a placeholder in this documentation:

```text
<EC2_PUBLIC_IP>
---

## 9. Validation

### TCP Connectivity Test

The current public IP was tested from Windows PowerShell:

```powershell
Test-NetConnection <EC2_PUBLIC_IP> -Port 80
TcpTestSucceeded : True


### HTTP Validation

The web server was then tested using:

```powershell
```curl http://<EC2_PUBLIC_IP>
` ```
The request returned:
```text
HTTP 200 OK
```
---

## 10. Troubleshooting Flow

The troubleshooting process followed a layered connectivity path:

```text
Windows Client
      |
      v
Current EC2 Public IP
      |
      v
Internet Gateway
      |
      v
VPC Route Table
      |
      v
Network ACL
      |
      v
Security Group
      |
      v
EC2 Instance
      |
      v
Host Firewall
      |
      v
Nginx
      |
      v
HTTP 200 OK
```

## 11. Key Commands

### Linux / Nginx

```bash
sudo ss -tulnp
curl http://localhost
sudo ufw status
sudo iptables -L -n -v
sudo nft list ruleset
```

### AWS Networking

Reviewed:

- EC2 Security Group rules
- VPC route table
- Network ACL rules
- EC2 public IPv4 address

### Windows Connectivity Testing

```powershell
Test-NetConnection <EC2_PUBLIC_IP> -Port 80
curl http://<EC2_PUBLIC_IP>
```
These commands were used to verify service availability, TCP connectivity, firewall configuration, and HTTP accessibility.

---

## 12. Lessons Learned

This troubleshooting exercise reinforced several important Cloud Support concepts:

- Always verify the application and listening port before changing network configuration.
- Troubleshoot connectivity systematically from the client toward the server.
- A successful local test does not guarantee external connectivity.
- AWS Security Groups, Network ACLs, VPC routing, and host firewalls must be considered separately.
- An EC2 instance's public IPv4 address can change after a stop/start cycle when an Elastic IP is not being used.
- Always verify that the client is connecting to the correct destination before changing firewall or network settings.
- Use evidence from each troubleshooting layer to eliminate possible causes rather than making unnecessary configuration changes.
- Validate the fix from the original client after making the correction.

---

## 13. Cloud Support Skills Demonstrated

This lab demonstrates practical experience with:

- TCP/IP and HTTP connectivity troubleshooting.
- Linux service and port verification.
- Nginx troubleshooting and validation.
- AWS EC2 networking.
- AWS Security Group analysis.
- VPC route table analysis.
- Network ACL analysis.
- Linux firewall troubleshooting with UFW, iptables, and nftables.
- Windows PowerShell network connectivity testing.
- Root cause analysis and systematic troubleshooting.
- Evidence-based problem isolation.
- Technical documentation and incident-style reporting.
- Security-conscious documentation using placeholders instead of sensitive infrastructure details.

---

## 14. Interview Talking Points

### Problem

An external HTTP connectivity test to an Nginx web server hosted on an AWS EC2 instance was initially failing.

### Investigation

I used a layered troubleshooting approach. I first verified that Nginx was running and that TCP port 80 was listening. I then tested the service locally and reviewed the EC2 Security Group, VPC route table, Network ACLs, UFW, iptables, and nftables.

### Root Cause

The server and network configuration were functioning correctly. The client was attempting to connect to an outdated EC2 public IPv4 address that had changed after the instance was stopped and restarted.

### Resolution

I identified the current EC2 public IPv4 address and repeated the connectivity test using the correct destination.

### Validation

`Test-NetConnection` confirmed successful TCP connectivity to port 80, and the HTTP request returned `HTTP 200 OK` with the expected Nginx webpage.

### Key Takeaway

The main lesson was to troubleshoot systematically and verify the destination before changing working network or firewall configurations.

---

## 15. Portfolio Context

This practical is part of my AWS Cloud Support / Cloud Engineer hands-on portfolio.

The lab demonstrates how I approach a real-world connectivity incident by:

- Identifying the reported symptom.
- Establishing a troubleshooting path.
- Collecting evidence from the application, operating system, and AWS networking layers.
- Eliminating potential causes systematically.
- Identifying the root cause.
- Applying the appropriate resolution.
- Validating the fix from the original client.
- Documenting the incident for future reference.

Future labs in this portfolio will build on these skills and cover additional areas such as AWS infrastructure, Linux administration, IAM, cloud monitoring, networking, ticket-based troubleshooting, automation, CI/CD, Infrastructure as Code, containers, and Kubernetes.

---

## Security and Privacy

This repository is intended to be publicly accessible and does not contain sensitive infrastructure information.

The documentation uses placeholders such as:

```text
<EC2_PUBLIC_IP>
```
