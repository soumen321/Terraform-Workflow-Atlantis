# Atlantis Integration with GitHub

## Prerequisites

* Amazon Linux 2023 EC2 Instance
* Docker Installed
* GitHub Personal Access Token (PAT)
* AWS Credentials with appropriate permissions
* Terraform Code Repository in GitHub

---

# Install Docker on Amazon Linux 2023

```bash
sudo yum update -y
sudo yum search docker -y
sudo yum install docker -y

sudo systemctl enable docker.service
sudo systemctl start docker.service
sudo systemctl status docker.service
```

Verify Docker installation:

```bash
docker --version
```

---

# Build Atlantis Docker Image

```bash
docker build -t atlantis .
```

---

# Run Atlantis Container

## Single Repository Access

```bash
docker run -itd \
-p 4000:4141 \
--name atlantis \
atlantis server \
--automerge \
--autoplan-modules \
--gh-user=<github-username> \
--gh-token=<github-personal-access-token> \
--repo-allowlist=github.com/<org-or-user>/<repository>
```

### Example

```bash
docker run -itd \
-p 4000:4141 \
--name atlantis \
atlantis server \
--automerge \
--autoplan-modules \
--gh-user=jerinrathnam \
--gh-token=<github-token> \
--repo-allowlist=github.com/easydeploy-cloud/terraform-atlantis
```

---

## Allow All Repositories in a GitHub Organization

```bash
docker run -itd \
-p 4000:4141 \
--name atlantis \
atlantis server \
--automerge \
--autoplan-modules \
--gh-user=<github-username> \
--gh-token=<github-token> \
--repo-allowlist=github.com/<organization>/*
```

### Example

```bash
docker run -itd \
-p 4000:4141 \
--name atlantis \
atlantis server \
--automerge \
--autoplan-modules \
--gh-user=jerinrathnam \
--gh-token=<github-token> \
--repo-allowlist=github.com/easydeploy-cloud/*
```

---

# Access Atlantis Container

Connect to the running Atlantis container:

```bash
docker exec -it atlantis /bin/sh
```

---

# Configure AWS Credentials

## Method 1: Using AWS CLI

```bash
aws configure
```

---

## Method 2: Manually Create Credentials File

Edit credentials file:

```bash
vi /home/atlantis/.aws/credentials
```

Add the following configuration:

```ini
[default]
aws_access_key_id = <ACCESS_KEY>
aws_secret_access_key = <SECRET_KEY>
```

Replace:

| Parameter  | Description               |
| ---------- | ------------------------- |
| ACCESS_KEY | AWS IAM Access Key ID     |
| SECRET_KEY | AWS IAM Secret Access Key |

### Multiple AWS Profiles

```ini
[default]
aws_access_key_id = XXXXX
aws_secret_access_key = XXXXX

[dev]
aws_access_key_id = XXXXX
aws_secret_access_key = XXXXX

[prod]
aws_access_key_id = XXXXX
aws_secret_access_key = XXXXX
```

Terraform can reference specific AWS profiles when required.

---

# Push Terraform Code to GitHub

```bash
git add .

git commit -m "Create EC2 Instance"

git checkout -b develop

git push origin develop
```

---

# Atlantis Commands

## Generate Terraform Plan

Comment on the Pull Request:

```text
atlantis plan
```

Or for a specific directory:

```text
atlantis plan -d .
```

---

## Apply Terraform Changes

Comment on the Pull Request:

```text
atlantis apply
```

Or for a specific directory:

```text
atlantis apply -d .
```

---

# Common Atlantis Workflow

```text
Developer creates PR
        │
        ▼
Atlantis automatically runs:
        │
        ▼
atlantis plan
        │
        ▼
Plan output posted to PR
        │
        ▼
Reviewer approves PR
        │
        ▼
User comments:
atlantis apply
        │
        ▼
Terraform changes applied
        │
        ▼
PR merged automatically (if --automerge enabled)
```

---

# Security Recommendations

❌ Do NOT pass GitHub tokens directly in Docker commands.

Instead, use environment variables:

```bash
docker run -itd \
-p 4000:4141 \
-e ATLANTIS_GH_USER=<github-user> \
-e ATLANTIS_GH_TOKEN=<github-token> \
--name atlantis \
atlantis server \
--automerge \
--autoplan-modules \
--repo-allowlist=github.com/<organization>/*
```

For AWS authentication, prefer:

* IAM Roles for EC2
* IAM Roles for Service Accounts (IRSA) on EKS
* AWS Secrets Manager

Avoid storing AWS access keys in plaintext files whenever possible.

---

# Useful Commands

Check running containers:

```bash
docker ps
```

View Atlantis logs:

```bash
docker logs -f atlantis
```

Restart Atlantis:

```bash
docker restart atlantis
```

Stop Atlantis:

```bash
docker stop atlantis
```

Remove Atlantis container:

```bash
docker rm -f atlantis
```

---

# References

* Atlantis Official Documentation: https://www.runatlantis.io/
* Terraform Documentation: https://developer.hashicorp.com/terraform/docs
* GitHub Actions Documentation: https://docs.github.com/en/actions
* Youtube Reference : https://www.youtube.com/watch?v=sV9IBczE3IA
