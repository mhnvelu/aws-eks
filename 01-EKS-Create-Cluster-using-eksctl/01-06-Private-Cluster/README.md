# Private Cluster
- No outbound internet access

## Requirements
If your cluster doesn't have outbound internet access, then it must meet the following requirements:
- Cluster must pull images from a **container registry** that's in your VPC
- Cluster must have **endpoint private access** enabled
- Cluster's **aws-auth ConfigMap** must be created from within your VPC
- Pods configured with IAM roles for service accounts acquire credentials from an AWS Security Token Service (AWS STS) API call. If there is no outbound internet access, you must create and use an **AWS STS VPC endpoint** in your VPC. Most AWS v1 SDKs use the global AWS STS endpoint by default (sts.amazonaws.com), which doesn't use the AWS STS VPC endpoint. To use the AWS STS VPC endpoint, you might need to configure your SDK to use the regional AWS STS endpoint (sts.region-code.amazonaws.com).
- Cluster's VPC subnets must have a **VPC interface endpoint** for any AWS services that your Pods need access to.
- For managed node group, the VPC interface endpoint security group must allow the CIDR for the subnets.
