## AWS EKS Core Objects

### EKS Cluster

#### EKS Control Plane

```
* Contains K8s master components like etcd, kube-apiserver, kube-controller.

* It's a managed service by AWS.

* EKS runs a single tenant Kubernetes control plane for each cluster, and control plane infrastructure is not shared across clusters or AWS accounts.

* This control plane consists of at least two API server nodes and three etcd nodes that run across three Availability Zones within a Region.

* EKS automatically detects and replaces unhealthy control plane instances, restarting them across the Availability Zones within the Region as needed.

```

#### Worker Nodes & Node Groups

```
* Group of EC2 instances where we run Application workloads.

* Nodes can be created either on PUBLIC or PRIVATE subnets.

* Worker machines in Kubernetes are called nodes.  These are EC2 Instances.

* EKS worker nodes run in our AWS account and connect to our cluster's control plane via the cluster API server endpoint.

* A node group is one or more EC2 instances that are deployed in an EC2 Autoscaling group.

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

### EKS Archiecture

- EKS Control Plane

  It contains same components as K8s control plane + Fargate Controller Manager, Fargate Scheduler

- EKS Managed Node Groups

  It contains group of Worker Nodes.

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
