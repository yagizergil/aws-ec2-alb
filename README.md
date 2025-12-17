# AWS Application Load Balancer + EC2 (Node.js) Production Lab

## Overview

This lab demonstrates a **production-grade AWS architecture** where a Node.js (Express) application running on an EC2 instance is exposed to the internet via an **Application Load Balancer (ALB)**.  
The setup includes **health checks, secure network isolation, and real-world debugging scenarios**.

The goal of this lab is to simulate how backend services are deployed and validated in real production environments.

---

## Architecture

Internet
|
v
Application Load Balancer (ALB)
|
v
Target Group (HTTP :3000)
|
v
EC2 (Ubuntu) → Node.js (Express App)



---

## Components Used

- **AWS EC2 (Ubuntu)**
- **Node.js (Express) application**
- **Application Load Balancer (ALB)**
- **Target Group (Instance-based)**
- **Security Groups**
- **Health Checks**

---

## Application Details

- Runtime: **Node.js**
- Framework: **Express**
- Application Port: **3000**
- Health Endpoint:  
GET /health


Response:
```json
{ "status": "healthy" }
Target Group Configuration
Setting	Value
Target type	Instance
Protocol	HTTP
Port	3000
Health check path	/health
Health check port	Traffic port
Interval	15 seconds
Healthy threshold	2
Unhealthy threshold	3
Success codes	200

Load Balancer Configuration
Type: Application Load Balancer

Scheme: Internet-facing

Listener:

HTTP :80 → Forward to target group

Deployed across multiple Availability Zones

Public DNS used for testing

Security Group Design (Production-Oriented)
ALB Security Group
Inbound:

HTTP (80) → 0.0.0.0/0

Outbound:

All traffic

EC2 Security Group
Inbound:

Custom TCP (3000) → ALB Security Group only

Outbound:

All traffic

This ensures the application port is not publicly accessible and can only be reached through the ALB.

Debugging & Incident Resolution
Issue Encountered
Target group remained UNHEALTHY after ALB attachment.

Root Cause
Express application was bound to 127.0.0.1 (localhost).

ALB cannot reach services bound to localhost.

Resolution
Updated the Express server to bind on 0.0.0.0:


app.listen(3000, '0.0.0.0');
Restarted the Node.js process.

Verified service accessibility via private IP.

Verification Commands

curl http://localhost:3000/health
curl http://<EC2_PRIVATE_IP>:3000/health
Result
Target state transitioned from UNHEALTHY → HEALTHY

ALB successfully routed traffic to EC2

Validation
Target Group status: Healthy

ALB → EC2 routing confirmed

Health checks passing consistently

Key Learnings
ALB health checks fail if applications are bound to localhost

Security Groups should reference other Security Groups, not public CIDRs

Always validate applications at:

localhost

private IP

load balancer level

Production debugging often requires application-level inspection, not only AWS resources

Next Steps
HTTPS with ACM on ALB

Domain routing (api.example.com → ALB)

Auto Scaling Group integration

ALB access logs to S3

Blue/Green deployments

Status
✅ Completed and production-ready
