# eks-sandbox

## Tutorial

https://www.youtube.com/watch?v=QThadS3Soig

## Set up AWS CLI

Run the custom CLI container

```
docker build -t my-aws-cli my-aws-cli
docker run -it --rm -v ${PWD}/work:/work -w /work --entrypoint /bin/bash my-aws-cli
```

Access keys

See the "Access keys" section in: https://console.aws.amazon.com/iam/home?region=us-west-2#/security_credentials

Region name
```
us-west-2
```

Login with the AWS CLI

```
aws configure
```

Look around

```
aws help
aws eks help
```

## Create an EKS cluster IAM role

Add policies to the role so that EKS can manage AWS resources, such as:
  * Load balancer
  * EC2 instances

```
aws iam create-role --role-name getting-started-eks-role --assume-role-policy-document file://assume-policy.json
```

Save the ARN

For the role to be able to do anything it needs a policy attached to it. The policy here is provided by AWS.

```
aws iam attach-role-policy --role-name getting-started-eks-role --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy
```

## Networking

See the file downloaded from https://amazon-eks.s3.us-west-2.amazonaws.com/cloudformation/2020-05-08/amazon-eks-vpc-sample.yaml into the `./work` directory.

Apply the network

```
aws cloudformation deploy --template-file amazon-eks-vpc-sample.yaml --stack-name getting-started-eks
```

This creates a CloudFormation stack named "getting-started-eks" which can be observed in the AWS console.

Get the `PhysicalResourceId`s of the three subnets

```
aws cloudformation list-stack-resources --stack-name getting-started-eks | grep subnet
```

Get the `PhysicalResourceId` of the security group

aws cloudformation list-stack-resources --stack-name getting-started-eks | grep -B 2 -A 8 ControlPlaneSecurityGroup

## Creating the cluster

Use the values obtained earlier

```
aws eks create-cluster --name getting-started-eks --role-arn arn:aws:iam::430780901527:role/getting-started-eks-role --resources-vpc-config subnetIds=<subnet1>,<subnet2>,<subnet3>,securityGroupIds=<security-group>,endpointPublicAccess=true,endpointPrivateAccess=false
```

Wait for cluster creation

```
aws eks list-clusters
aws eks describe-cluster --name getting-started-eks
```

## .kube/config

```
aws eks update-kubeconfig --name getting-started-eks --region us-west-2
kubectl version
kubectl get nodes
```

## EC2 instances

There are currently no nodes so another role needs to manage "node groups" that we can attach EC2 instances to and deploy into the VPC.

Create a new role and make note of the `Arn`:

```
aws iam create-role --role-name getting-started-eks-role-nodes --assume-role-policy-document file://assume-node-policy.json
```

Add three policies for this role to be able to (respectively)

* manage the nodes
* container networking in the node group
* ability to pull from a container registry within the account

```
aws iam attach-role-policy --role-name getting-started-eks-role-nodes --policy-arn arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy
aws iam attach-role-policy --role-name getting-started-eks-role-nodes --policy-arn arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy
aws iam attach-role-policy --role-name getting-started-eks-role-nodes --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly
```

Start some t3.small instances

```
aws eks create-nodegroup \
--cluster-name getting-started-eks \
--nodegroup-name test \
--node-role <role-arn> \
--subnets <one-of-the-subnets> \
--disk-size 200 \
--scaling-config minSize=1,maxSize=2,desiredSize=1 \
--instance-types t2.small
```

# Example app

```
kubectl create ns example-app
kubectl apply -n example-app -f example-app/k8s/deployment.yml
kubectl apply -n example-app -f example-app/k8s/service.yml

kubectl get svc -n example-app
```

App will be available at the `LoadBalancer Ingress` value, which looks like `abcdefghijklmnopqrstuvwxyz.us-west-2.elb.amazonaws.com`

# Clean up

https://docs.aws.amazon.com/eks/latest/userguide/delete-cluster.html

Delete in reverse order:
* The service, because it has an `EXTERNAL-IP` value associated with an AWS load balancer
* The node group
* The cluster
* The cluster role
* The cluster node roles
* The networking stack

```
kubectl delete svc example-service -n example-app

aws eks delete-nodegroup --nodegroup-name test --cluster-name getting-started-eks
aws eks delete-cluster --name getting-started-eks

aws iam detach-role-policy --role-name getting-started-eks-role --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy
aws iam delete-role --role-name getting-started-eks-role

aws iam detach-role-policy --role-name getting-started-eks-role-nodes --policy-arn arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy
aws iam detach-role-policy --role-name getting-started-eks-role-nodes --policy-arn arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy
aws iam detach-role-policy --role-name getting-started-eks-role-nodes --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly
aws iam delete-role --role-name getting-started-eks-role-nodes

aws cloudformation list-stacks --query "StackSummaries[].StackName"
aws cloudformation delete-stack --stack-name getting-started-eks
```
