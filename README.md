# CloudNova AWS Infrastructure

## Project Overview
This project demonstrates a 3-tier e-commerce application deployed on AWS using separate App, Operations, and Data VPCs.

The solution focuses on network isolation, high availability, shared storage, automated backups, monitoring, and recovery.

## Architecture

```text
                         Customers
                             |
                             v
                    +------------------+
                    |    Public ALB    |
                    |      HTTP :80    |
                    +------------------+
                             |
                             v
                    +------------------+
                    | Presentation Tier|
                    | Private EC2      |
                    | Nginx + Frontend |
                    +------------------+
                             |
                             v
                    +------------------+
                    |   Internal ALB   |
                    |      HTTP :80    |
                    +------------------+
                             |
                             v
                    +------------------+
                    |     App Tier     |
                    | Private EC2      |
                    | Node.js Backend  |
                    +------------------+
                             |
                             v
                    +------------------+
                    |  RDS PostgreSQL  |
                    |   Private DB     |
                    +------------------+

   Operations VPC                  Data VPC
   +----------------+              +----------------+
   |  Bastion Host  |              | Backup EC2     |
   | Admin Access   |              | Bash + Cron    |
   +----------------+              +----------------+
           \                              |
            \                             v
             +---- Transit Gateway ----> S3 Backups

   App Tier ------> EFS Shared Storage

   CloudWatch ----> SNS Notifications
   Auto Scaling --> Unhealthy Instance Replacement
```

## How the Solution Works

1. **Network isolation:** Three VPCs separate application resources, administration access, and backup resources. A Transit Gateway connects the required VPCs, while route tables and security groups control traffic.

2. **Customer requests:** The public Application Load Balancer receives HTTP requests and forwards them to Presentation instances in private subnets.

3. **Presentation tier:** Nginx serves the frontend and forwards `/api/` requests to the internal Application Load Balancer.

4. **Application tier:** The internal ALB distributes requests to private EC2 instances running the Node.js backend.

5. **Database:** The backend connects to private Amazon RDS for PostgreSQL to read and manage e-commerce data.

6. **Shared storage:** Amazon EFS allows application instances to access shared files across Availability Zones.

7. **Backups:** An EC2 instance in the Data VPC creates compressed backups using Bash and uploads them to Amazon S3 on a scheduled cron job through an S3 Gateway VPC Endpoint.

8. **Monitoring and recovery:** CloudWatch alarms monitor unhealthy load balancer targets and send notifications through SNS. Auto Scaling Groups can replace unhealthy instances when their configured health checks detect failures.

## AWS Services

- **Networking:** VPC, Subnets, Route Tables, Transit Gateway
- **Compute & Load Balancing:** EC2, Application Load Balancer, Auto Scaling Groups
- **Database & Storage:** RDS PostgreSQL, EFS, S3
- **Security:** IAM, Security Groups, Bastion Host, VPC Endpoints
- **Monitoring:** CloudWatch, SNS
- **Automation:** Bash, Cron, systemd


## Repository Structure

```text
CloudNova-NTI/
├── README.md
├── scenario/
│   └── CloudNova_Scenario.pdf
├── docs/
│   └── CloudNova_Documentation.pdf
├── diagrams/
│   └── architecture.png
└── screenshots/
    ├── 01-vpcs-and-subnets.png
    ├── 02-transit-gateway.png
    ├── 03-bastion-host.png
    ├── 04-rds-postgresql.png
    ├── 05-public-alb.png
    ├── 06-internal-alb.png
    ├── 07-auto-scaling-groups.png
    ├── 08-efs-shared-storage.png
    ├── 09-s3-backup.png
    ├── 10-cloudwatch-alarms.png
    ├── 11-sns-notifications.png
    └── 12-self-healing-test.png
```



## Validation

- Accessed the application through the public ALB.
- Verified frontend-to-backend routing.
- Tested PostgreSQL connectivity.
- Verified shared files through EFS.
- Tested S3 backup uploads.
- Configured CloudWatch alarms and SNS notifications.
- Observed Auto Scaling replace an unhealthy EC2 instance.

## Documentation

See [`docs/CloudNova_Documentation.pdf`](docs/CloudNova_Documentation.pdf) for detailed implementation steps, commands, configurations, and testing.

## Notes

This is an AWS learning project. The demo uses HTTP, and lab resources should be removed after testing to avoid unnecessary charges.
