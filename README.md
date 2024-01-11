# VProfile Application (Lift & Shift Cloud Deployment)

## 📋 Project Overview

The **VProfile** project is a multi-tier Java web application that demonstrates a **Lift & Shift** migration strategy to AWS. This approach involves moving an existing on-premises application to the cloud with minimal modifications, enabling rapid cloud adoption while preserving the existing application logic and structure.

## 🏗️ Architecture

### Components

The application consists of four main tiers:

1. **Web Tier (Load Balancer)**

   - AWS Application Load Balancer (ALB)
   - Handles HTTPS traffic with ACM certificates
   - Distributes requests across application instances

2. **Application Tier**

   - Tomcat application server running on Ubuntu EC2 (`app01`)
   - Hosts the VProfile Java web application
   - Auto-scales based on demand
   - Communicates with backend services via Route53 private DNS

3. **Backend Services Tier**

   - **MySQL Database** (`db01`) - Amazon Linux 2023
   - **Memcached Cache** (`mc01`) - Amazon Linux 2023
   - **RabbitMQ Message Queue** (`rmq01`) - Amazon Linux 2023

4. **DNS & Service Discovery**
   - Route53 Private Hosted Zone (`vprofile.in`)
   - Internal DNS resolution for backend services
   - Enables service discovery without hardcoded IP addresses

## 🚀 Deployment Steps

### Prerequisites

- AWS Account with appropriate permissions
- Local machine with Maven, AWS CLI, and SSH client
- Domain name (for DNS configuration)
- VProfile application source code (vprofile_app)

### Step-by-Step Deployment

1. **[Security Groups & Key Pairs](deployment/cloud/01_securitygroups_&_keypairs.md)**

   - Create `vprofile-elb-sg` for load balancer
   - Create `vprofile-app-sg` for application tier
   - Create `vprofile-backend-sg` for backend services
   - Generate RSA key pair (`vprofile-keypair.pem`)

2. **[EC2 Instances](deployment/cloud/02_ec2_instances.md)**

   - Launch `db01` - MySQL database server
   - Launch `mc01` - Memcached cache server
   - Launch `rmq01` - RabbitMQ message broker
   - Launch `app01` - Tomcat application server
   - All instances use automated setup via user data scripts

3. **[Route53 Private DNS](deployment/cloud/03_dns_route53.md)**

   - Create private hosted zone: `vprofile.in`
   - Configure DNS records for internal service discovery:
     - `db01.vprofile.in` → Database
     - `mc01.vprofile.in` → Cache
     - `rmq01.vprofile.in` → Message Queue
     - `app01.vprofile.in` → Application (for health checks)

4. **[Build & Deploy Artifact](deployment/cloud/04_build_&_deploy_artifact.md)**

   - Create S3 bucket for artifact storage
   - Build application: `mvn install`
   - Upload WAR file to S3
   - Deploy artifact to Tomcat on `app01`
   - Create IAM roles for EC2 → S3 access

5. **[Load Balancer & DNS](deployment/cloud/05_loadbalancer_&_dns.md)**

   - Create target group pointing to `app01` on port 8080
   - Request SSL certificate via AWS ACM
   - Create Application Load Balancer (ALB)
   - Map domain name to ALB endpoint via CNAME record

6. **[Auto Scaling Setup](deployment/cloud/06_autoscaling_groups.md)**

   - Create AMI from configured `app01` instance
   - Create launch template for application tier
   - Configure Auto Scaling Group (min: 1, max: 2)
   - Set scaling policy based on CPU utilization (50% target)
   - Enable load balancer stickiness for session persistence

7. **Verification**
   - Access application via load balancer URL
   - Verify SSL/HTTPS certificate
   - Test database connectivity
   - Validate caching layer
   - Confirm message queue functionality

## 📁 Project Structure

```
vprofile_app/

├── pom.xml                 # Maven configuration
├── src/
│   ├── main/              # Application source code
│   └── test/              # Test cases
├── target/                # Build artifacts
└── userdata/              # Automated setup scripts
    ├── mysql.sh           # Database initialization
    ├── memcache.sh        # Cache server setup
    ├── rabbitmq.sh        # Message queue setup
    └── tomcat_ubuntu.sh   # Application server setup

deployment/
├── cloud/                 # AWS cloud deployment docs
│   ├── 00_app_deployment_cloud.md
│   ├── 01_securitygroups_&_keypairs.md
│   ├── 02_ec2_instances.md
│   ├── 03_dns_route53.md
│   ├── 04_build_&_deploy_artifact.md
│   ├── 05_loadbalancer_&_dns.md
│   ├── 06_autoscaling_groups.md
│   └── 07_vprofile_live.md
├── local/                 # Local deployment docs
└── resources/             # Documentation resources
    ├── credentials/       # AWS credentials
    └── Images/           # Architecture & configuration images
```

## 🔧 VProfile Technology Stack

### Required Software

- **JDK**: 11
- **Build Tool**: Maven 3
- **Application Server**: Tomcat
- **Database**: MySQL 8

### Core Technologies

- Spring MVC
- Spring Security
- Spring Data JPA
- Maven
- JSP
- Tomcat
- MySQL
- Memcached
- RabbitMQ
- Elasticsearch

## 🗄️ Database Setup

This project uses MySQL. Import the database dump file located at `src/main/resources/db_backup.sql`:

```bash
mysql -u <username> -p accounts < db_backup.sql
```

Alternative method using the application properties configuration:

```bash
mysql -u root -p < src/main/resources/accountsdb.sql
```

## 🔐 Security Configuration

### Security Groups

| Name                  | Purpose            | Inbound Rules                                                |
| --------------------- | ------------------ | ------------------------------------------------------------ |
| `vprofile-elb-sg`     | Load Balancer      | HTTP (80), HTTPS (443) from anywhere                         |
| `vprofile-app-sg`     | Application Server | HTTP (8080) from ELB, SSH (22) from admin IP                 |
| `vprofile-backend-sg` | Backend Services   | MySQL (3306), Memcached (11211), RabbitMQ (5672) from app-sg |

### Network

- **Private Hosted Zone**: `vprofile.in` (internal DNS only)
- **SSL/TLS**: AWS ACM certificates for HTTPS
- **IAM Roles**: EC2 instances have minimal S3 access for artifact deployment

## 🛠️ Building & Deploying

### Local Build

```bash
cd vprofile_app
mvn clean install
```

### Deploy to AWS

1. **Configure AWS CLI**

```bash
aws configure
# Enter Access Key, Secret Key, Region (us-east-1), Output (yaml)
```

1. **Upload Artifact to S3**

```bash
aws s3 cp vprofile_app/target/vprofile-v2.war s3://vprofile-artifact-v01/
```

1. **SSH to app01 and Deploy**

```bash
ssh -i vprofile-keypair.pem ubuntu@<ALB-DNS-Name>
sudo -i
aws s3 cp s3://vprofile-artifact-v01/vprofile-v2.war /tmp/
systemctl stop tomcat10
cp /tmp/vprofile-v2.war /var/lib/tomcat10/webapps/ROOT.war
systemctl start tomcat10
```

## 📊 Application Access

Once deployed:

- **Web Application**: `https://yourdomain.com` (via ALB)
- **Direct Access**: `http://<app01-public-ip>:8080` (during testing)
- **Health Checks**: Route53 DNS resolution for backend services

## 🔄 Auto Scaling

The Auto Scaling Group monitors CPU utilization and automatically scales the application tier:

- **Minimum Instances**: 1
- **Maximum Instances**: 2
- **Target CPU Utilization**: 50%
- **Cooldown Period**: 300 seconds

## 🎯 Benefits of Lift & Shift

✅ **Minimal Code Changes** - No application refactoring required  
✅ **Fast Migration** - Quicker time-to-cloud deployment  
✅ **Cost Reduction** - Eliminate on-premises infrastructure  
✅ **Scalability** - Leverage auto-scaling and load balancing  
✅ **High Availability** - Multi-AZ deployment across regions  
✅ **Security** - AWS managed security services (ACM, Security Groups)

## 📖 Documentation

Comprehensive deployment guides are available in [`deployment/cloud`](deployment/cloud):

- Architecture overview: 00_app_deployment_cloud.md
- Security setup: 01*securitygroups*&\_keypairs.md
- Instance creation: 02_ec2_instances.md
- DNS configuration: 03_dns_route53.md
- Artifact deployment: 04*build*&\_deploy_artifact.md
- Load balancer setup: 05*loadbalancer*&\_dns.md
- Auto scaling: 06_autoscaling_groups.md
- Live verification: 07_vprofile_live.md

## 📝 License

This project demonstrates DevOps and cloud deployment practices for educational purposes.

---

**Last Updated**: 2025 | **Project Type**: Lift & Shift Cloud Migration | **Cloud Provider**: AWS
