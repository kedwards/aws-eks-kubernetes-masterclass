# EKS - Create EKS Node Group in Private Subnets

## Step-01: Introduction
- We are going to create a node group in VPC Private Subnets
- We are going to deploy workloads on the private node group wherein workloads will be running private subnets and load balancer gets created in public subnet and accessible via internet.

```sh
# Create Cluster
eksctl create cluster --name=eksrtls1 \
  --region=us-west-2 \
  --zones=us-west-2a,us-west-2b \
  --without-nodegroup

# Get List of clusters
eksctl get cluster
```

## Step-02: Delete existing Public Node Group in EKS Cluster
```
# Get NodeGroups in a EKS Cluster
eksctl get nodegroup --cluster=eksrtls1

# Delete Node Group - Replace nodegroup name and cluster name
eksctl delete nodegroup eksrtls1-ng-public1 --cluster eksrtls1
```

## Step-03: Create EKS Node Group in Private Subnets
- Create Private Node Group in a Cluster
- Key option for the command is `--node-private-networking`

```
eksctl create nodegroup --cluster=eksrtls1 \
  --region=us-west-2 \
  --name=eksrtls1-ng-private1 \
  --node-type=t3.medium \
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
  --alb-ingress-access \
  --node-private-networking
```

## Step-04: Verify if Node Group created in Private Subnets

### Verify External IP Address for Worker Nodes
- External IP Address should be none if our Worker Nodes created in Private Subnets
```
kubectl get nodes -o wide
```
### Subnet Route Table Verification - Outbound Traffic goes via NAT Gateway
- Verify the node group subnet routes to ensure it created in private subnets
  - Go to Services -> EKS -> eksrtls -> eksrtls1-ng1-private
  - Click on Associated subnet in **Details** tab
  - Click on **Route Table** Tab.
  - We should see that internet route via NAT Gateway (0.0.0.0/0 -> nat-xxxxxxxx)
