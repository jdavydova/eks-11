### EKS AWS Kubernetes

https://docs.aws.amazon.com/eks/latest/userguide/creating-a-vpc.html

Amazon S3 url bucket https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2020-10-29/amazon-eks-vpc-private-subnets.yaml


	aws configure list 
 	aws eks update-kubeconfig --name eks-cluster-test 

Cluster status is CREATING

To find latest version for autoscaler

https://github.com/kubernetes/autoscaler/tags

Added to  cluster-autoscaler-autodiscaver.yaml

————————————
name: cluster-autoscaler
namespace: kube-system
annotations
eks.amazonaws.com/role-arn: arn:aws:iam::788577008603:role/EKSServiceAccountRole
————————————————
containers:
- image: registry.k8s.io/autoscaling/cluster-autoscaler:v1.35.0
  name: cluster-autoscaler
  env:
    - name: AWS_REGION
      value: "eu-north-1"
—————————————————
--node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/<EKS_CLUSTRER_NAME>
- --balance-similar-node-groups
- --skip-nodes-with-system-pods=false
————————————————


    kubectl apply -f cluster-autoscaler-autodiscaver.yaml 

## Fargate

### Create IAM Role for Fargate

EC2 Role for Node Group (Worker Node)

1. Pods/kebectl on server provisioned by Fargate need permissions
2. Create Fargate Role 

IAM -> Role -> AWS Service -> EKS Fargate Pod

Role Name :
    eks-fargate-role

### Create Fargate Profile 

1. Pod Selection Rule
2. Specifies which Pods Should use Fargarte when they are lounched 

eks-cluster-test -> Computer -> Add fargate profile

    kubectl get pod 
    kubectl get node -n kube-system
    kubectl get ns
    kubectl create ns dev
    kubectl apply -f fargate-profile/nginx-config.yaml 
    kubectl get pods -n dev

To install eksctl

    brew tap weaveworks/tap
    brew install weaveworks/tap/eksct

EKS create cluster:

    eksctl create cluster \
    --name demo-cluster \
    --version 1.35 \
    --region eu-north-1 \
    --nodegroup-name demo-nodes \
    --node-type t3.micro \
    --nodes 2 \
    --nodes-min 1 \
    --nodes-max 3