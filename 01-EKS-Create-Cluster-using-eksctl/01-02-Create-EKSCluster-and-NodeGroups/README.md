# Create EKS Cluster & Node Groups

## Step-00: Introduction
- Understand about EKS Core Objects
  - Control Plane
  - Worker Nodes & Node Groups
  - Fargate Profiles
  - VPC
- Create EKS Cluster
- Associate EKS Cluster to IAM OIDC Provider
- Create EKS Node Groups
- Verify Cluster, Node Groups, EC2 Instances, IAM Policies and Node Groups


## Step-01: Create EKS Cluster using eksctl
- It will take 15 to 20 minutes to create the Cluster Control Plane

```sh
# Create Cluster
eksctl create cluster --name=eksrtls1 \
  --region=us-west-2 \
  --zones=us-west-2a,us-west-2b \
  --without-nodegroup

# Get List of clusters
eksctl get cluster
```

## Step-02: Create & Associate IAM OIDC Provider for our EKS Cluster
- To enable and use AWS IAM roles for Kubernetes service accounts on our EKS cluster, we must create &  associate OIDC identity provider.
- To do so using `eksctl` we can use the  below command.
- Use latest eksctl version (as on today the latest version is `0.21.0`)

```sh
eksctl utils associate-iam-oidc-provider \
  --region us-west-2 \
  --cluster eksrtls1 \
  --approve
```
## Step-03: Create EC2 Keypair
- Create a new EC2 Keypair
- This keypair we will use it when creating the EKS NodeGroup.
- This will help us to login to the EKS Worker Nodes using Terminal.

```
aws ec2 create-key-pair --key-name eksrtls1
```

## Step-04: Create Node Group with additional Add-Ons in Public Subnets
- These add-ons will create the respective IAM policies for us automatically within our Node Group role.
 ```
# Create Public Node Group
eksctl create nodegroup --cluster=eksrtls1 \
  --region=us-west-2 \
  --name=eksrtls1-ng-public1 \
  --node-type=t3.medium \
  --nodes=2 \
  --nodes-min=2 \
  --nodes-max=4 \
  --node-volume-size=20 \
  --ssh-access \
  --ssh-public-key=eksrtls \
  --managed \
  --asg-access \
  --external-dns-access \
  --full-ecr-access \
  --appmesh-access \
  --alb-ingress-access
```

[Private node Groups](../../07-ELB-Classic-and-Network-LoadBalancers/07-01-Create-EKS-Private-NodeGroup/README.md)

## Step-05: Verify Cluster & Nodes

### Verify NodeGroup subnets to confirm EC2 Instances are in Public Subnet
- Verify the node group subnet to ensure it created in public subnets
  - Go to Services -> EKS -> eksrtls1 -> eksrtls1-ng1-public
  - Click on Associated subnet in **Details** tab
  - Click on **Route Table** Tab.
  - We should see that internet route via Internet Gateway (0.0.0.0/0 -> igw-xxxxxxxx)

### Verify Cluster, NodeGroup in EKS Management Console
- Go to Services -> Elastic Kubernetes Service -> eksrtls1



```sh
# set context
aws eks --region us-west-2 update-kubeconfig --name eksrtls1
```
### List Worker Nodes
```sh
# List EKS clusters
eksctl get cluster

# List NodeGroups in a cluster
eksctl get nodegroup --cluster=eksrtls1

# List Nodes in current kubernetes cluster
kubectl get nodes -o wide

# Our kubectl context should be automatically changed to new cluster
kubectl config view --minify
```

### Verify Worker Node IAM Role and list of Policies
- Go to Services -> EC2 -> Worker Nodes
- Click on **IAM Role associated to EC2 Worker Nodes**

### Verify Security Group Associated to Worker Nodes
- Go to Services -> EC2 -> Worker Nodes
- Click on **Security Group** associated to EC2 Instance which contains `remote` in the name.

### Verify CloudFormation Stacks
- Verify Control Plane Stack & Events
- Verify NodeGroup Stack & Events

### Login to Worker Node using Keypair
- Login to worker node
```
# For MAC or Linux or Windows10
ssh -i eksrtls.pem ec2-user@<Public-IP-of-Worker-Node>

# For Windows 7
Use putty
```

## Step-06: Update Worker Nodes Security Group to allow all traffic
- We need to allow `All Traffic` on worker node security group

## Additional References
- https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html
- https://docs.aws.amazon.com/eks/latest/userguide/create-service-account-iam-policy-and-role.html
- https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_create_oidc.html
- https://docs.aws.amazon.com/eks/latest/userguide/create-kubeconfig.html
