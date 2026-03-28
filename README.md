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
###### AWS Fargate = serverless compute for containers
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

### 11 - Kubernetes on AWS - EKS

Exercises for Module "Kubernetes on AWS"


Right after you setup the cluster on LKE or Minikube and deployed your application inside, your manager comes to you to tell you that the company also wants to run Kubernetes on AWS. Again, with less overhead when managing just one platform. So they ask you to reconfigure your cluster on AWS and deploy your application there instead.

🟢 EXERCISE 1: Create EKS cluster
You decide to create an EKS cluster - AWS managed Kubernetes Service. To simplify the whole creation and configuration process, you use eksctl.

With eksctl you create an EKS cluster with 3 Nodes and 1 Fargate profile

🔹 Install tools (if not installed)
	
	brew install eksctl
	brew install kubectl
	brew install awscli

Configure AWS:

	aws configure
	
🔹 Create EKS cluster (3 nodes)
	eksctl create cluster \
	  --name demo-cluster \
	  --region eu-north-1 \
	  --nodes 3

🔹 Step 3 — Verify cluster

	kubectl get nodes

🔹 Step 4 — Create Fargate profile

	eksctl create fargateprofile \
	  --cluster demo-cluster \
	  --region eu-north-1 \
	  --name demo-fargate \
	  --namespace default
  
🔹 Step 5 — Deploy application

Example (nginx):

	kubectl create deployment nginx --image=nginx
	kubectl expose deployment nginx --type=LoadBalancer --port=80
	
🔹 Step 6 — Verify deployment

	kubectl get pods
	kubectl get svc

👉 Look for EXTERNAL-IP

Using configuration file :

	kubectl apply -f nginx-fargate.yaml

Example of nginx-fargate.yaml:

	apiVersion: apps/v1
	kind: Deployment
	metadata:
	  name: nginx-test
	  namespace: default
	  labels:
	    app: nginx
	spec:
	  replicas: 2
	  selector: 
	    matchLabels:
	      app: nginx
	      profile: fargate
	  template:
	    metadata:
	      labels:
	        app: nginx
	        profile: fargate
	    spec:
	      containers:
	      - name: nginx
	        image: nginx
	        ports:
	        - containerPort: 80 
	---
	apiVersion: v1
	kind: Service
	metadata:
	  name: nginx
	  labels:
	    app: nginx
	spec:
	  ports:
	  - name: http
	    port: 80
	    protocol: TCP
	    targetPort: 80
	  selector:
	    app: nginx
	  type: LoadBalancer
	  
🔹 Step 7 — Access application

Open in browser:

	http://<EXTERNAL-IP>

You should see nginx welcome page ✅

⚠️ VERY IMPORTANT (cost control)
🔥 Step 8 — Cleanup (MUST DO)
eksctl delete cluster \
  --name demo-cluster \
  --region eu-north-1

This removes:

cluster
nodes (EC2)
load balancers
networking


🟢 EXERCISE 2: Deploy Mysql and phpmyadmin
You deploy mysql and phpmyadmin on the EC2 nodes using the same setup as before.
﻿NOTE: Bitnami have recently changed their repository structure and have moved their MySQL helm charts to the "bitnamilegacy" repository. Add the following to your values YAML file in order to use the correct Helm chart:

yaml
global:
  security:
    allowInsecureImages: True
image:
  registry: docker.io
  tag: latest
  repository: bitnamilegacy/mysql 
EXERCISE 3: Deploy your Java application
You deploy your Java application using Fargate with 3 replicas using the same setup as before.



Setup Continuous Deployment with Jenkins
EXERCISE 4: Automate deployment
Now your application is running, and when you or others make changes to it, Jenkins pipeline builds the new image, but you have to manually deploy it into the cluster. From your experience you know how annoying that is for you and your team, so you want to automate deploying to the cluster as well.

Setup automatic deployment to the cluster in the pipeline.


EXERCISE 5: Use ECR as Docker repository
Now your application is running, and when you or others make changes to it, Jenkins pipeline builds the new image, but you have to manually deploy it into the cluster. From your experience you know how annoying that is for you and your team, so you want to automate deploying to the cluster as well.

So, the company wants to use ECR instead, again to have everything on 1 platform and also to let AWS manage the repository including storage, cleanups etc. Therefore you:

Replace the docker repository in your pipeline with ECR

EXERCISE 6: Configure Autoscaling
Now your application is running, whenever a change is made, it gets automatically deployed in the cluster etc. This is great, but you notice that most of the time the 3 nodes you have are underutilized, especially at the weekends, because your containers aren't using that many resources. However, your company is still paying full price for all of the servers.

You suggest to your manager that you will be able to save the company some infrastructure costs by configuring autoscaling. Your manager is happy with this suggestion and asks you to configure it. So you:

Go ahead and configure autoscaling to scale down to a minimum of 1 node when servers are underutilized and maximum of 3 nodes when in full use.



