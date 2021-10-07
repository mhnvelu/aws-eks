## AWS EKS

- Managed service that you can use to run Kubernetes on AWS without needing to install, operate, and maintain your own Kubernetes control plane or nodes.
- Runs and scales the `Kubernetes control plane across multiple AWS Availability Zones` to ensure high availability.
- `Automatically scales control plane` instances based on load, detects and replaces unhealthy control plane instances, and it provides automated version updates and patching for them.
- `Integrated with many AWS services` to provide scalability and security for your applications, including the following capabilities:
  - Amazon ECR for container images
  - Elastic Load Balancing for load distribution
  - IAM for authentication
  - Amazon VPC for isolation

## AWS EKS Core Objects

### EKS Cluster

#### EKS Control Plane

```
* Contains K8s master components like etcd, kube-apiserver, kube-controller.

* It's a managed service by AWS.

* EKS runs a single tenant Kubernetes control plane for each cluster, and control plane infrastructure is not shared across clusters or AWS accounts.

* This control plane consists of at least two API server nodes and three etcd nodes that run across three Availability Zones within a Region.

* EKS automatically detects and replaces unhealthy control plane instances, restarting them across the Availability Zones within the Region as needed.

* Amazon VPC network policies to restrict traffic between control plane components to within a single cluster.


```

#### Worker Nodes & Node Groups

```
* Group of EC2 instances where we run Application workloads.

* Nodes can be created either on PUBLIC or PRIVATE subnets.

* Worker machines in Kubernetes are called nodes.  These are EC2 Instances.

* EKS worker nodes run in our AWS account and connect to our cluster's control plane via the cluster API server endpoint.

* A node group is one or more EC2 instances that are deployed in an EC2 Autoscaling group.

* Any number of nodegroups can be created under a single cluster.

* All instances in a node group must
    - Be the same instance type
    - Be running the same AMI
    - Use the same EKS worker node IAM role

```

#### Fargate Profiles (Serverless)

      ```
      * Instead of EC2 instances, we can run Application workloads on Fargate.

      * Fargate profiles can be created only if we have atleast ONE PRIVATE subnet in the VPC.

      * AWS Fargate is a technology that provides on-demand, right-sized compute capacity for containers.

      * With Fargate, we no longer have to provision, configure, or scale groups of virtual machines to run containers. 

      * Each pod running on Fargate has its own isolation boundary.

      * AWS specially built Fargate controllers that recognizes the pods belonging to fargate and schedules them on Fargate profiles.

#### VPC

```
* With AWS VPC we follow secure networking standards which will allow us to run Application workloads on EKS

* Workloads deployed on private subnet should be able to communicate with EKS Control Plane. So we need to deploy VPC NAT gateway.

* EKS uses AWS VPC network policies to restrict traffic between control plane components to within a single cluster.

* Control plane components for a EKS cluster cannot view or receive communication from other clusters or other AWS accounts, except as authorized with Kubernetes RBAC policies.

* This secure and highly-available configuration makes EKS reliable and recommended for production workloads.

```

#### Amazon EKS cluster endpoint

- Amazon EKS creates an `endpoint for the managed Kubernetes API server` that you use to communicate with your cluster.
- By default, this API server endpoint is `public to the internet`, and access to the API server `is secured` using a combination of AWS Identity and Access Management (IAM) and native Kubernetes Role Based Access Control (RBAC)

  ```
  Because this endpoint is for the Kubernetes API server and not a traditional AWS PrivateLink endpoint for communicating with an AWS API, it doesn't appear as an endpoint in the Amazon VPC console.
  ```

- You `can enable private access to the Kubernetes API server` so that all communication between your nodes and the API server stays within your VPC. **Preferred for Production.**
- You can limit the IP addresses that can access your API server from the internet, or completely disable internet access to the API server.
- You can define your API server endpoint access requirements when you create a new cluster, and you can update the API server endpoint access for a cluster at any time.

- Accessing a private only API server:
  - Connected network - Connect your network to the VPC with an AWS transit gateway or other connectivity option and then use a computer in the connected network.
  - Amazon EC2 bastion host
  - AWS Cloud9 IDE

#### Cluster Autoscaler

- Automatically adjusts the number of nodes in your cluster when pods scheduling fails or are rescheduled onto other nodes.
- Managed node groups are implemented using Amazon EC2 Auto Scaling groups, and are compatible with the Cluster Autoscaler.
- Managed node groups are implemented using Amazon EC2 Auto Scaling groups, and are compatible with the Cluster Autoscaler.
- If your existing node groups were created with eksctl and you used the `--asg-access` option, then this policy already exists.

### EKS Archiecture

- EKS Control Plane

  - It contains same components as K8s control plane + Fargate Controller Manager, Fargate Scheduler.
  - Each Amazon EKS cluster control plane is single-tenant and unique, and runs on its own set of Amazon EC2 instances.
  - The Kubernetes API is exposed via the Amazon EKS endpoint associated with your cluster.
  - `The control plane runs in an account managed by AWS`.
  - All of the data stored by the etcd nodes and associated Amazon EBS volumes is encrypted using AWS KMS.
  - The cluster control plane is provisioned across multiple Availability Zones and fronted by an Elastic Load Balancing Network Load Balancer.
  - Amazon EKS also provisions elastic network interfaces in your VPC subnets to provide connectivity from the control plane instances to the nodes

- EKS Managed Node Groups

  - It contains group of Worker Nodes.
  - Amazon `EKS nodes run in your AWS account` and connect to your cluster's control plane via the API server endpoint and a certificate file that is created for your cluster.

- Worker Node

  It runs Container Runtime, Kubelet, Kube-Proxy

### EKS Storage

- In Tree EBS Provisioner
- EBS CSI Driver
- EFS CSI Driver
- FSx for Luster CSI
- These drivers are not supported for AWS EKS Fargate
- These drivers allow EKS clusters to `manage lifecycle` of EBS volumes for persistent storage, EFS file systems & FSx for Luster File Systems

#### AWS EBS

- Provides `block level storage volumes` for use with `EC2 & container instances`
- EBS volumes that are attached to an instance are exposed as storage volumes that persist independently from the life of the EC2 or Container instances.
- The configuration of the volume can be changed dynamically.
- AWS recommends EBS for data that must be `quickly accessible` and requires `long-term persistence`.
- EBS is well suited for `database-type applications` that rely on random reads and writes and to `through-put instensive applications` that perform long continuous reads and writes.

#### Drawbacks of EBS for DB

- Complex setup to achieve HA
- Complex Multi-AZ support for EBS. EBS is a zone level resource which means the volumes are zone restricted and can't be across the entire region.
- Complex Master-Master DB setup
- Complex Master-Workers DB setup
- No automatic backup & recovery
- No auto-upgrade of DB

#### EKS cluster + AWS RDS

- Using k8s ExternalName service, the k8s cluster can connect to AWS RDS service.
- Features of AWS RDS:
  - Automatic Backup & Recovery
  - HA
  - Read Replicas
  - Automatic upgrades
  - Metrics & Monitoring
  - Multi-AZ support
