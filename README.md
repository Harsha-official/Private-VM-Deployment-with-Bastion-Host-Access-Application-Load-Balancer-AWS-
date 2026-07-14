---------------------------------------------------------------------------------

                                    DEMO

                                    


---------------------------------------------------------------------------------

# Private VM Deployment with Bastion Host Access & Application Load Balancer (AWS)

## Project Overview

This project demonstrates deploying an application on an **EC2 instance in a
private subnet** (no direct internet access), securely accessing it via a
**Bastion Host**, and exposing the application to the internet through an
**Application Load Balancer (ALB)**.

The goal was to follow AWS best practices for network security — keeping
application servers private while still allowing controlled administrative
access and public application access.

---

##  Architecture

```
Internet
   │
   ▼
┌─────────────────────────────┐
│   Application Load Balancer │  (Public Subnet, internet-facing)
│         :8080               │
└─────────────┬───────────────┘
              │
              ▼
┌─────────────────────────────┐
│     Private EC2 Instance     │  (Private Subnet)
│   Application running :8080  │
└─────────────────────────────┘
              ▲
              │ SSH (via Bastion only)
              │
┌─────────────────────────────┐
│        Bastion Host          │  (Public Subnet)
└─────────────────────────────┘
              ▲
              │ SSH
              │
          Admin / Developer
```

- **VPC** with public and private subnets across two Availability Zones
  (`us-east-1a`, `us-east-1b`)
- **Bastion Host** in the public subnet — the only entry point for SSH access
- **Private EC2 instance** with no public IP, reachable only via the bastion
- **Application Load Balancer** (internet-facing) routing public traffic to
  the private instance
- **Target Group** health-checking the private instance

---

##  What Was Done

1. Created a VPC with public and private subnets across two AZs.
2. Launched a **Bastion Host** in the public subnet with a restricted
   security group (SSH access from a trusted IP only).
3. Launched a **private EC2 instance** (no public IP) inside the private
   subnet.
4. Connected to the private instance **through the bastion host** using SSH
   agent forwarding / a jump-host configuration.
5. Deployed the application on the private instance, running on port `8080`.
6. Created an **Application Load Balancer** in the public subnets with a
   listener on port `8080`, forwarding to a target group containing the
   private instance.
7. Configured security groups so that:
   - The **ALB's security group** allows inbound traffic on `8080` from the
     internet.
   - The **instance's security group** allows inbound traffic on `8080`
     only from the ALB's security group, and SSH only from the bastion's
     security group.
8. Verified target health in the target group (`Healthy`).
9. Verified end-to-end access by browsing to the ALB's public DNS name and
   confirming the application loads.

---

##  Result

- Target group shows the private instance as **Healthy**.
- Application is successfully deployed and accessible via the ALB's public
  DNS name on port `8080`.
- The application server itself is never directly exposed to the internet —
  all public traffic passes through the ALB, and all administrative access
  passes through the bastion host.

**Access URL pattern:**
```
http://<alb-dns-name>:8080
```

---

##  Security Notes

- No public IP is assigned to the application instance.
- SSH access to the private instance is only possible by first connecting
  to the bastion host.
- Security groups follow least-privilege: each layer only accepts traffic
  from the specific layer in front of it (Internet → ALB → Instance,
  Admin → Bastion → Instance).

---

## Evidence

*(Add your screenshots here, e.g.)*
- Target group showing healthy target
- ALB listener configuration
- Application accessible via ALB DNS name
- Bastion host SSH session into the private instance

---

## Tools & Services Used

- AWS VPC (public/private subnets, route tables)
- AWS EC2 (Bastion Host + private application instance)
- AWS Application Load Balancer (ALB)
- AWS Security Groups
- SSH (bastion jump-host access)
