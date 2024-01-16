# 🎒 S3 Bucket, IAM Roles & Artifact Deployment

---

This section covers creating an S3 bucket for storing build artifacts, configuring IAM access, and deploying the WAR file to the Tomcat application server.

---

### 🪣 1️⃣ Create S3 Bucket

**Bucket Details**

- **Name:** `vprofile-artifact`
- **Object Ownership:** ACLs disabled
- **Block Public Access:** Enabled
- **Versioning:** Disabled
- **Region:** `us-east-1 (N. Virginia)`
- **Bucket Type:** General Purpose

![alt text](../resources/Images/vprofile-s3.png)

---

### 🔐 2️⃣ Create IAM user for CLI Upload

#### Role: `vprofile-s3-admin-user`

- Policy Type: _Attach existing policies directly_
- Policy: **AmazonS3FullAccess**

![vprofile-s3-admin-user](../resources/Images/vprofile-s3-admin-user.png)

#### Create Access Keys

- Use case: **Command Line Interface**
- Download CSV containing:

  - Access Key ID
  - Secret Access Key

![vprofile-s3-admin-user-key](../resources/Images/vprofile-s3-admin-user-key.png)

![vprofile-s3-admin-user-key_2](../resources/Images/vprofile-s3-admin-user-key_2.png)

You’ll use these credentials to upload artifacts from your local machine.

---

### 🔐 3️⃣ Create IAM Role for EC2 Access to S3

#### Role: `vprofile-ec2-s3-access-role`

- **Trusted Entity:** AWS Service
- **Use Case:** EC2
- **Attached Policy:** `AmazonS3FullAccess`

![vprofile-ec2-s3-access-role](../resources/Images/vprofile-ec2-s3-access-role.png)

**Note** this role name—will be applied to `app01` instance.

---

### 🖇️ 4️⃣ Attach IAM Role to `app01`

**EC2 → app01 → Actions → Security → Modify IAM Role**

Select:

```
vprofile-ec2-s3-access-role
```

Update role assignment.

![attach_iam_role_ec2](../resources/Images/attach_iam_role_ec2.png)

---

### 📦 5️⃣ Build & Upload Artifact to S3

In VS Code terminal:

#### Verify Dependencies

```bash
mvn -v               # checks Maven availability
aws --version        # checks AWS CLI availability
```

![alt text](../resources/Images/check-dependencies.png)

#### Build the Artifact

```bash
mvn install
```

![app_build_1](../resources/Images/app_build_1.png)

A WAR file will appear in the `target/` directory.

![app_build_2](../resources/Images/app_build_2.png)

---

#### Configure AWS CLI

```bash
aws configure
```

Enter:

- Access Key ID
- Secret Access Key
- Region: `us-east-1`
- Output Format: `YAML`

![aws-configuration](../resources/Images/aws-configuration.png)

---

#### Upload WAR File to S3

```bash
aws aws s3 cp target/vprofile-v2.war s3://vprofile-artifact-v01/
aws s3 ls s3://vprofile-artifact-v01/
```

Verify the file is uploaded.

![artifact_upload_s3](../resources/Images/artifact_upload_s3.png)

---

### 🧩 6️⃣ Deploy Artifact to Tomcat on EC2

#### Log in to `app01`

```bash
ssh -i vprofile-keypair.pem ubuntu@<EC2-Public-IP>
sudo -i
```

![login_app01](../resources/Images/login_app01.png)

---

#### Install AWS CLI (if not installed)

```bash
snap install aws-cli --classic
aws --version
```

![aws-cli_in_app01](../resources/Images/aws-cli_in_app01.png)

---

#### Download Artifact from S3

```bash
aws s3 cp s3://vprofile-artifact-v01/vprofile-v2.war /tmp/
```

![copy_artifact_ec2t](../resources/Images/copy_artifact_ec2.png)

---

#### Stop Tomcat

```bash
systemctl stop tomcat10
```

![stop_tomcat10](../resources/Images/stop_tomcat10.png)

#### Remove Existing ROOT App

```bash
ls /var/lib/tomcat10/webapps
rm -rf /var/lib/tomcat10/webapps/ROOT
```

![remove_root](../resources/Images/remove_root.png)

#### Deploy the New WAR File

```bash
cp /tmp/vprofile-v2.war /var/lib/tomcat10/webapps/ROOT.war
```

![replace_root](../resources/Images/replace_root.png)

#### Start Tomcat

```bash
systemctl start tomcat10
```

![start_tomcat10](../resources/Images/start_tomcat10.png)

---
