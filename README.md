# AWS Web Hosting with Ansible

## Overview
This project sets up a highly available and secure web hosting environment on AWS. The architecture consists of a Virtual Private Cloud (VPC) with public and private subnets across two Availability Zones (AZs). It leverages an Application Load Balancer (ALB) for traffic distribution and SSL/TLS encryption to secure traffic in transit.

## Architecture Diagram

```
                      Internet
                          |
                          ▼
                 +-------------------+
                 |   Route 53 (DNS)  |
                 +-------------------+
                          |
                          ▼
            +----------------------------+
            | Application Load Balancer  | (Public Subnet - AZ1 & AZ2)
            +----------------------------+
                          |
                  +-------+-------+
                  |               |
    +-------------------+   +-------------------+
    | App Server (AZ1)  |   | App Server (AZ2)  | (Private Subnet)
    +-------------------+   +-------------------+
                  ▲               ▲
                  |               |
             +---------+-----+----------+
             |       Ansible  Server    | (Private Subnet)
             +---------+-----+----------+               
                          ▲
                          |
             +---------+-----+----------+
             |       Bastion Host       | (Public Subnet)
             +---------+-----+----------+               
       
```

## Components

### 1. Virtual Private Cloud (VPC)
- **Subnets**: 
  - Public Subnet (for the Bastion Host, ALB, and NAT Gateway) 
  - Private Subnet (for Ansible and Application Servers)
- **Availability Zones**: Two AZs for high availability.

### 2. Internet Gateway
- Provides internet access to resources in public subnets.

### 3. NAT Gateway
- Allows instances in private subnets to access the internet while remaining unreachable from the outside.

### 4. Bastion Host
- Provides secure SSH access to instances in the private subnet.

### 5. Ansible Server
- Used to automate and manage the configuration of the application servers.

### 6. Application Servers
- Hosts the website, configured by Ansible.
- Placed in private subnets for security.

### 7. Application Load Balancer (ALB)
- Accepts incoming traffic and routes it to the application servers.
- Configured to use HTTPS for secure communication.

### 8. SSL Certificate
- Issued by AWS Certificate Manager (ACM) to secure traffic in transit.
- Associated with the ALB to enable HTTPS.

### 9. Route Tables
- **Public Route Table**: Routes internet-bound traffic to the Internet Gateway.
- **Private Route Table**: Routes outbound traffic from private subnets to the internet through NAT Gateway.

### 10. Security Groups
- **Bastion Host SG**: Allows SSH access from trusted IPs.
- **ALB SG**: Allows HTTP/HTTPS access from the internet.
- **Application Servers SG**: Allows HTTP/HTTPS traffic only from the ALB. Allows  SSH access from Ansible Server. 
- **Ansible Server SG**: Restricted to SSH access from the bastion host.

## Configuration Steps

1. **VPC Setup:**
   - Create a VPC with CIDR block (e.g., 10.0.0.0/16).
   - Create two public subnets and two private subnets across different AZs.

2. **Internet and NAT Gateway Setup:**
   - Attach an Internet Gateway to the VPC.
   - Create a NAT Gateway in a public subnet and associate an Elastic IP.

3. **Route Table Configuration:**
   - Associate the Internet Gateway with the public route table.
   - Associate the NAT Gateway with the private route table for outbound traffic.

4. **Bastion Host Setup:**
   - Launch an EC2 instance in the public subnet for SSH access.
   - Associate an Elastic IP with the bastion host.

5. **Ansible Server Setup:**
   - Launch an EC2 instance in the private subnet.
   - Use the bastion host to SSH into the Ansible server.

6. **Application Servers Setup:**
   - Launch two EC2 instances in private subnets.
   - Use Ansible to configure and deploy the website.

7. **Application Load Balancer Setup:**
   - Create an ALB in the public subnets.
   - Configure target groups to point to the application servers.
   - Enable HTTPS by attaching the SSL certificate from ACM.

8. **DNS Configuration:**
   - Create a new record in Route 53 to point to the ALB.

9. **SSL Certificate:**
   - Request a certificate via AWS Certificate Manager (ACM).
   - Validate the certificate using DNS and associate it with the ALB.

## Security Considerations
- **Access Control**: Restrict SSH access to the bastion host only.
- **Private Subnet Isolation**: Ensure application servers are not exposed to the internet.
- **SSL Encryption**: Encrypt all traffic using HTTPS.
- **Least Privilege**: Apply the principle of least privilege in IAM policies.

## Accessing the Website
1. Ensure the website domain is configured in **Route 53**.
2. Use the HTTPS endpoint to securely access the application.

## Future Improvements
- Implement Auto Scaling for application servers.
- Enable logging and monitoring using CloudWatch.
- Use AWS WAF to protect against common web exploits.

## Troubleshooting
1. **SSL Certificate Issues**: Ensure the CNAME records are correctly added in Route 53.
2. **Access Issues**: Check the security group rules for the ALB and EC2 instances.
3. **Website Not Loading**: Verify the health check configuration for the ALB target group.
4. **SSH Agent Forwarding Issues**: Ensure agent forwarding is enabled on your local machine and bastion host:
- Start SSH agent and add your key:
```bash
eval $(ssh-agent)
ssh-add /path/to/private-key.pem
```
- Use the -A flag when connecting to the bastion host to enable agent forwarding:
```bash 
ssh -A -i /path/to/private-key.pem ec2-user@your-public-bastion-host-ip
```
- Confirm agent forwarding is active on the bastion host:
```bash
ssh-add -L
```
- Now, SSH into the private EC2 from the public EC2 without needing the key file:
```bash
ssh ubuntu@your-private-ansible-server-ip
```

## Conclusion
This setup provides a robust, secure, and scalable web hosting environment on AWS. By using a bastion host, private subnets, and an ALB with SSL termination, the architecture ensures secure access and high availability.


