# Amazon EKS cluster endpoint access control
- EKS creates an endpoint for the managed Kubernetes API server that you use to communicate with your cluster
- By default, this API server endpoint is public to the internet, and access to the API server is secured using a combination of AWS Identity and Access Management (IAM) and native Kubernetes Role Based Access Control (RBAC).
- There 3 options for endpoint access control
    - Public only access
        - Default behavior
        - Node to control plane communication leave the VPC but not Amazon's network
        - API server is accessible from the internet
        - Optionally, limit the CIDR blocks that can access the public endpoint
    - Both Public and Private access enabled
        - Node to control plane communication use the private VPC endpoint
        - API server is accessible from the internet
        - Optionally, limit the CIDR blocks that can access the public endpoint
    - Private only access
        - Communication between your nodes and the API server stays within your VPC
        - Amazon EKS creates a **Route 53 private hosted zone** on your behalf and associates it with your cluster's VPC. This private hosted zone is managed by Amazon EKS, and it doesn't appear in your account's Route 53 resources
        - In order for the private hosted zone to properly route traffic to your API server, your VPC must have **enableDnsHostnames and enableDnsSupport set to true**, and the DHCP options set for your VPC must include AmazonProvidedDNS in its domain name servers list
        - Traffic to API server must come from within your cluster's VPC or a connected network.
        - No public access to your API server from the internet

# Accessing a private only API server
For all the options mentioned below, we need to make sure  Amazon EKS control plane security group contains rules to allow ingress traffic on port 443
- Connected network
- Amazon EC2 bastion host
- AWS Cloud9 IDE