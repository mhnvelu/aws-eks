# Install AWS, kubectl & eksctl CLI's

2 Options available:

- eksctl - simple command line utility for creating and managing Kubernetes clusters on Amazon EKS.
- AWS managment console and AWS CLI.

## Step-00: Introduction

- Install AWS CLI
- Install kubectl CLI
- Install eksctl CLI

## Step-01: Install AWS CLI

- Reference-1: https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html
- Reference-2: https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html

### Step-01-01: Windows 10 - Install and configure AWS CLI

- The AWS CLI version 2 is supported on Windows XP or later.
- The AWS CLI version 2 supports only 64-bit versions of Windows.
- Download Binary: https://awscli.amazonaws.com/AWSCLIV2.msi
- Install the downloaded binary (standard windows install)

```
aws --version
aws-cli/2.0.57 Python/3.7.7 Windows/10 exe/AMD64
```

- Reference: https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-windows.html

### Step-01-02: Configure AWS Command Line using Security Credentials

- Go to AWS Management Console --> Services --> IAM
- Create an IAM user : admin-user
- Attach policy : AdministratorAccess
- **Important Note:** Use only IAM user to generate **Security Credentials**. Never ever use Root User. (Highly not recommended)
- Once the admin-user is created successfully, Copy Access ID and Secret access key
- Go to command line and provide the required details

```
aws configure
AWS Access Key ID [None]: xxxxxxx  (Replace your creds when prompted)
AWS Secret Access Key [None]: xxxxxxxxxxxxxxx  (Replace your creds when prompted)
Default region name [None]: eu-west-1
Default output format [None]: json
```

- Test if AWS CLI is working after configuring the above

```
aws ec2 describe-vpcs
```

## Step-02: Install kubectl CLI

- **IMPORTANT NOTE:** Kubectl binaries for EKS please prefer to use from Amazon (**Amazon EKS-vended kubectl binary**)
- This will help us to get the exact Kubectl client version based on our EKS Cluster version. You can use the below documentation link to download the binary.
- Reference: https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html

### Step-02-01: Windows 10 - Install and configure kubectl

- Install kubectl on Windows 10

```
mkdir kubectlbinary
cd kubectlbinary
curl -o kubectl.exe https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/windows/amd64/kubectl.exe
```

- Update the system **Path** environment variable

```
C:\Users\<your-name>\Documents\kubectlbinary
```

- Verify the kubectl client version

```
kubectl version --short --client
Client Version: v1.19.6-eks-49a6c0

kubectl version --client
```

## Step-03: Install eksctl CLI

### Step-03-01: eksctl on windows or linux

- For windows and linux OS, you can refer below documentation link.
- **Reference:** https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html#installing-eksctl

## References:

- https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html
