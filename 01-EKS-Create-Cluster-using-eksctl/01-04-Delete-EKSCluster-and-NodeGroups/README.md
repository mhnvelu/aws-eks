# Delete EKS Cluster & Node Groups

- When you're done using an Amazon EKS cluster, you should delete the resources associated with it so that you don't incur any unnecessary costs.
- If you have active services in your cluster that are associated with a load balancer, you must delete those services before deleting the cluster so that the load balancers are deleted properly. Otherwise, you can have orphaned resources in your VPC that prevent you from being able to delete the VPC.

## Step-01: Delete Services

Delete any services that have an associated EXTERNAL-IP value. These services are fronted by an Elastic Load Balancing load balancer, and you must delete them in Kubernetes to allow the load balancer and associated resources to be properly released.

```
kubectl get svc --all-namespaces
kubectl delete svc <service-name>
```

## Step-02: Delete Node Group

- We can delete a nodegroup separately using below `eksctl delete nodegroup`

```
# List EKS Clusters
eksctl get clusters

# Capture Node Group name
eksctl get nodegroup --cluster=<clusterName>
eksctl get nodegroup --cluster=eksdemo1

# Delete Node Group
eksctl delete nodegroup --cluster=<clusterName> --name=<nodegroupName>
eksctl delete nodegroup --cluster=eksdemo1 --name=eksdemo1-ng-public1
```

## Step-03: Delete Cluster

- We can delete cluster using `eksctl delete cluster`

```
# Delete Cluster
eksctl delete cluster <clusterName>
eksctl delete cluster eksdemo1
```

## Important Notes

### Note-1: Rollback any Security Group Changes

- When we create a EKS cluster using `eksctl` it creates the worker node security group with only port 22 access.
- When we progress through the course, we will be creating many **NodePort Services** to access and test our applications via browser.
- During this process, we need to add an additional rule to this automatically created security group, allowing access to our applications we have deployed.
- So the point we need to understand here is when we are deleting the cluster using `eksctl`, its core components should be in same state which means roll back the change we have done to security group before deleting the cluster.
- In this way, cluster will get deleted without any issues, else we might have issues and we need to refer cloudformation events and manually delete few things. In short, we need to go to many places for deletions.

### Note-2: Rollback any EC2 Worker Node Instance Role - Policy changes

- When we are doing `EBS Storage Section with EBS CSI Driver` we will add a custom policy to worker node IAM role.
- When you are deleting the cluster, first roll back that change and delete it.
- This way we don't face any issues during cluster deletion.
