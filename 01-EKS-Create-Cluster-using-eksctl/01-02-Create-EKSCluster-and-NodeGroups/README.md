# Create EKS Cluster & Node Groups

## Step-00: Introduction

- Create EKS Cluster
- Associate EKS Cluster to IAM OIDC Provider
- Create EKS Node Groups
- Verify Cluster, Node Groups, EC2 Instances, IAM Policies and Node Groups

## Step-01: Create EKS Cluster using eksctl

- It will take 15 to 20 minutes to create the Cluster Control Plane

```
# Create Cluster
eksctl create cluster --name=eksdemo1 \
                      --region=eu-west-1 \
                      --zones=eu-west-1a,eu-west-1b \
                      --without-nodegroup

# Get List of clusters
eksctl get clusters
```

- This will not create worker nodes. It will create only the EKS Control Plane on the specified zones.
- By default, API endpoints are available for public access.
- Once the cluster is created, the kubeconfig file is automatically created and saved locally. Any kubectl and eksctl commands will use this kubeconfig file.

## Step-02: Create & Associate IAM OIDC Provider for our EKS Cluster

- To enable and use AWS IAM roles for Kubernetes service accounts on our EKS cluster, we must create & associate OIDC identity provider.
- To do so using `eksctl` we can use the below command.
- Use latest eksctl version

```
# Template
eksctl utils associate-iam-oidc-provider \
    --region region-code \
    --cluster <cluster-name> \
    --approve

# Replace with region & cluster name
eksctl utils associate-iam-oidc-provider \
    --region eu-west-1 \
    --cluster eksdemo1 \
    --approve
```

## Step-03: Create EC2 Keypair

- Create a new EC2 Keypair with name as `kube-demo`
  ```
  aws ec2 create-key-pair --region eu-west-1 --key-name kube-demo
  ```
- This keypair will be used when creating the EKS NodeGroup.
- This will help us to login to the EKS Worker Nodes using Terminal.

## Step-04: Create Node Group with additional Add-Ons in Public Subnets

- These add-ons will create the respective IAM policies for us automatically within our Node Group role.

```
# Create Public Node Group
eksctl create nodegroup --cluster=eksdemo1 \
                       --region=eu-west-1 \
                       --name=eksdemo1-ng-public1 \
                       --node-type=t3.medium \
                       --nodes=2 \
                       --nodes-min=2 \
                       --nodes-max=4 \
                       --node-volume-size=20 \
                       --ssh-access \
                       --ssh-public-key=kube-demo \
                       --managed \
                       --asg-access \
                       --external-dns-access \
                       --full-ecr-access \
                       --appmesh-access \
                       --alb-ingress-access
```

## Step-05: Verify Cluster & Nodes

### Verify NodeGroup subnets to confirm EC2 Instances are in Public Subnet

- Verify the node group subnet to ensure it created in public subnets
  - Go to Services -> EKS -> eksdemo -> eksdemo1-ng1-public
  - Click on Associated subnet in **Details** tab
  - Click on **Route Table** Tab.
  - We should see that internet route via Internet Gateway (0.0.0.0/0 -> igw-xxxxxxxx)

### Verify Cluster, NodeGroup in EKS Management Console

- Go to Services -> Elastic Kubernetes Service -> eksdemo1

### List Worker Nodes

```
# List EKS clusters
eksctl get cluster

# List NodeGroups in a cluster
eksctl get nodegroup --cluster=<clusterName>

# List Nodes in current kubernetes cluster
kubectl get nodes -o wide

# Our kubectl context should be automatically changed to new cluster
kubectl config view --minify
```

### Verify Worker Node IAM Role and list of Policies

- Go to Services -> EC2 -> Worker Nodes
- Click on **IAM Role associated to EC2 Worker Nodes**
- The policies included are related to ECR, Worker Node, etc.
- The pods running on these Worker Nodes can access these AWS services.
- The other way is create IAM role, assign policies, create k8s service account and assocaite to IAM role. Then assign the service account to Pods.

### Verify Security Group Associated to Worker Nodes

- Go to Services -> EC2 -> Worker Nodes
- Click on **Security Group** associated to EC2 Instance which contains `remote` in the name.
- By default it has only one rule and allows only SSH on port 22.

### Verify NAT gateway is created

- If the workloads run on Private subnet, then they should be able to communicate with EKS Control Plane and the Pods also need to access external services like docker hub.
- Go to Services -> VPC -> NAT Gateways
- One NAT gateway should have been created.
- We can create HA NAT gateway or disable this NAT gateway as well.

### Verify CloudFormation Stacks

- eksctl commands result in a creation of Cloud Formation Template and it creates the resources.
- Verify Control Plane Stack & Events
- Verify NodeGroup Stack & Events

### Login to Worker Node using Keypair kube-demo

- Login to worker node

```
# For MAC or Linux or Windows10
ssh -i kube-demo.pem ec2-user@<Public-IP-of-Worker-Node>

# For Windows 7
Use putty
```

## Step-06: Update Worker Nodes Security Group to allow all traffic

- We need to allow `All Traffic` on worker node security group
- This is needed when we use NodePort ServiceType for our workloads on EKS cluster. NodePort service generates different dynamic ports. So we can access the application as `WORKER_NODE_PUBLIC_IP:NODE_PORT`

## IAM entity as K8s cluster administrator

- The IAM entity (user or role) that created the cluster in above steps is added to the Kubernetes RBAC authorization table as the administrator (with system:masters permissions).
- Initially, only that IAM user can make calls to the Kubernetes API server using kubectl. If you want other users to have access to your cluster, then you must add them to the `aws-auth ConfigMap`.
- Restrict access to IMDS : Assign IAM roles to all of your Kubernetes service accounts so that pods only have the minimum permissions that they need, and no pods in the cluster require access to the Amazon EC2 instance metadata service (IMDS) for other reasons.

## Additional References

- https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html
- https://docs.aws.amazon.com/eks/latest/userguide/create-service-account-iam-policy-and-role.html
