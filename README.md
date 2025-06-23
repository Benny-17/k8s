# Kops Kubernetes Demo on AWS

This project demonstrates how to set up a Kubernetes cluster using Kops on AWS EC2. It's designed as a learning exercise for DevOps and cloud practitioners.

> ⚠️ Note: The cluster was not fully provisioned (`kops update cluster --yes` was intentionally skipped) to avoid AWS Free Tier costs. However, the full configuration flow was completed including Kops setup and S3 integration.

---

## 📌 Objective

- Set up Kops CLI on an Ubuntu EC2 instance
- Create a Kubernetes cluster configuration with Kops
- Store cluster state in an S3 bucket
- Understand how Kops provisions resources

---

## ⚙️ Prerequisites

- AWS account with IAM user (with S3 & EC2 permissions)
- Ubuntu EC2 instance (t2.micro for demo)
- AWS CLI configured via `aws configure`

---

## 📦 Tools Used

| Tool       | Purpose                           |
|------------|-----------------------------------|
| AWS CLI    | Configure AWS and create S3 bucket |
| kubectl    | Kubernetes command-line tool       |
| Kops       | Create and manage Kubernetes clusters |
| EC2 + S3   | Infrastructure for demo            |

---

## 🪜 Setup Steps

### 1. SSH into EC2 and Install Dependencies

```bash
sudo apt update -y
sudo apt install -y python3 awscli curl unzip
```
### 2. Install kubectl
```bash
curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```
### 3. Install Kops
```bash
curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x kops-linux-amd64
sudo mv kops-linux-amd64 /usr/local/bin/kops
```
### 4. Create an S3 Bucket (for Kops state)
```bash
aws s3api create-bucket \
  --bucket kops-demo-storage \
  --region us-east-1
```
### 5. Create the Kubernetes Cluster Config
```bash
kops create cluster \
  --name=demok8scluster.k8s.local \
  --state=s3://kops-demo-storage \
  --zones=us-east-1a \
  --node-count=1 \
  --node-size=t2.micro \
  --master-size=t2.micro \
  --master-volume-size=8 \
  --node-volume-size=8
```
📝 Using .k8s.local as a DNS suffix for testing/demo purposes.

❌ Stopping Before Provisioning
To avoid incurring charges, we stopped at this point. We didn’t run:

```bash
kops update cluster --yes
```
At this stage, only the cluster definition is stored in the S3 bucket. No infrastructure was created.

🧯 Optional: Clean-Up
If you want to delete the cluster config and the S3 bucket:
```bash
# Delete cluster config from S3
kops delete cluster --name=demok8scluster.k8s.local --state=s3://kops-demo-storage --yes
# Delete S3 bucket
aws s3 rb s3://kops-demo-storage --force
```

✅ What's Next (If Fully Proceeding)
If you wanted to actually build the cluster (note: this will incur AWS charges):
```bash
# Edit cluster configuration if needed
kops edit cluster demok8scluster.k8s.local --state=s3://kops-demo-storage

# Launch the cluster
kops update cluster demok8scluster.k8s.local --yes --state=s3://kops-demo-storage

# Validate the cluster
kops validate cluster --state=s3://kops-demo-storage
```
