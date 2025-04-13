# AWS Architecture with Bastion, Load Balancers, EC2, and RDS


## 📊 Architecture Diagram
![secuirty group flow with other components](terraform-aws-secuirty-group\images\sg_drawio.svg)

## Overview

This document outlines the architecture and traffic flow for a secure and scalable AWS-based web application. It includes public and private-facing components, using load balancers, bastion host access, and database access control.

---

## 🔧 Components

- **Frontend Load Balancer (FrontendLB)** - Public-facing
- **Frontend EC2 Instances (frontend-servers)** - Private
- **Backend Load Balancer (BackendLB)** - Internal-facing
- **Backend EC2 Instances (backend-servers)** - Private
- **Amazon RDS (MySQL Database)** - Private
- **Bastion Host** - Admin Access
- **Employees** - SSH and troubleshooting
- **End Users** - Application access

---

## 🔐 Security Group Rules & Flow

### 1. Users ➝ Frontend Load Balancer
- Public access
- **Allowed ports**: `80 (HTTP)`, `443 (HTTPS)`

### 2. Frontend Load Balancer ➝ Frontend EC2 Instances
- Internal access only
- **Allowed ports**: `80`
- ❌ Do not allow other ports

### 3. Frontend EC2 ➝ Backend Load Balancer
- Internal routing (e.g., DNS like `backend-dev.aws.online`)
- Use **port 80** for clean URL routing
- **Allowed ports**: `80`

### 4. Backend Load Balancer ➝ Backend EC2 Instances
- Internal load balancing
- **Allowed ports**: `8080`

### 5. Backend EC2 ➝ Amazon RDS
- Database access (MySQL)
- **Allowed ports**: `3306`
- RDS is private and not load balanced (managed by AWS)
- Accessible via **RDS endpoint**

---

## 👨‍💻 Bastion Host Access

### Purpose
Used by employees for secure access to private instances for troubleshooting.

### Access Rules

#### From Employees ➝ Bastion
- **Allowed ports**: `22` (SSH)

#### From Bastion ➝ Frontend EC2
- **Allowed ports**: `22`, `80` (SSH & HTTP app check)

#### From Bastion ➝ Backend Load Balancer
- **Allowed port**: `80` (No SSH access)

#### From Bastion ➝ Backend EC2
- **Allowed ports**: `22`, `8080`

#### From Bastion ➝ RDS (MySQL)
- **Allowed port**: `3306`

---

## 🧠 Notes

- **Frontend Load Balancer** is **public-facing** (Internet-facing)
- **Backend Load Balancer** is **private-facing** (Internal only)
- **RDS** does not use a load balancer, access is managed via RDS **endpoint**
- Bastion host acts as the **only jump box** for all private access
- All other resources reside in **private subnets**

---

## 🔁 Traffic Flow Summary

1. **User** accesses the app via `FrontendLB` on port `80/443`
2. Traffic goes to **Frontend EC2** on port `80`
3. Frontend server calls internal **BackendLB** on port `80`
4. BackendLB routes traffic to **Backend EC2** on port `8080`
5. Backend EC2 connects to **RDS** on port `3306` for DB queries
6. **Employees** use **Bastion** to troubleshoot backend/frontend or DB as needed

---

## 📌 Best Practices

- Use **Security Groups** to strictly control access to each component
- Keep **Backend Load Balancer** internal to avoid exposing it publicly
- Monitor access logs and restrict Bastion access to only authorized employees
- Rotate credentials and use IAM roles wherever possible

