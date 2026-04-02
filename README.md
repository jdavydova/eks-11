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
	  --nodes 3 \
	  --kubeconfig=./kubeconfig.my-cluster.yaml

🔹 Step 3 — Verify cluster

	kubectl get nodes

🔹 Step 4 — Create Fargate profile

	eksctl create fargateprofile \
	  --cluster demo-cluster \
	  --region eu-north-1 \
	  --name demo-fargate \
	  --namespace my-app
  
🔹 Step 5 — Deploy application

Example (nginx):

	kubectl create deployment nginx --image=nginx
	kubectl expose deployment nginx --type=LoadBalancer --port=80
	
🔹 Step 6 — Verify deployment

	kubectl get pods
	kubectl get svc

👉 Look for EXTERNAL-IP
	  
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

#### Solutions from techword:
First you need to install eksctl command line tool locally. See the installation guide here: https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html

Steps
🔹 create cluster with 3 EC2 instances and store access configuration to cluster in kubeconfig.my-cluster.yaml file 
	
	eksctl create cluster --name=my-cluster --nodes=3 --kubeconfig=./kubeconfig.my-cluster.yaml

 🔹 create fargate profile in the cluster. It will apply for all K8s components in my-app namespace

	eksctl create fargateprofile \
	    --cluster my-cluster \
	    --name my-fargate-profile \
	    --namespace my-app

🔹 point kubectl to your cluster - use absolute path to kubeconfigfile

	export KUBECONFIG={absolute-path}/kubeconfig.my-cluster.yaml

🔹 validate cluster is accessible and nodes and fargate profile created

	kubectl get node
	eksctl get fargateprofile --cluster my-cluster

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

General notes

All the k8s manifest files for the exercise are in "k8s-deployment" folder, so:

🔹 clone this repository locally

	git clone https://gitlab.com/twn-devops-bootcamp/latest/11-eks/eks-exercises.git

🔹 check out the solutions branch

	git checkout feature/solutions

🔹 change to k8s-deployment folder

	cd k8s-deployment

🔹Mysql Chart link:
	https://github.com/bitnami/charts/tree/master/bitnami/mysql

🔹 install Mysql chart 

	helm repo add bitnami https://charts.bitnami.com/bitnami
	helm install my-release -f mysql-chart-values-eks.yaml

🔹 deploy phpmyadmin with its configuration for Mysql DB access

	kubectl apply -f db-config.yaml
	kubectl apply -f db-secret.yaml
	kubectl apply -f phpmyadmin.yaml

🔹 access phpmyadmin and login to mysql db

	kubectl port-forward svc/phpmyadmin-service 8081:8081

🔹 access in browser on
	
	localhost:8081

🔹 login with one of these 2 credentials
	
	"my-user" : "my-pass"
	"root" : "secret-root-pass"


### Installing phpMyAdmin Locally (Minikube + MySQL + Kubernetes)

🔹 Step 1 — Start local Kubernetes cluster
minikube start

👉 This creates a local Kubernetes cluster on your machine.

🔹 Step 2 — Install MySQL using Helm

Add Helm repository:

	helm repo add bitnami https://charts.bitnami.com/bitnami

Install MySQL:

	helm install my-release bitnami/mysql -f mysql-chart-values-eks.yaml

👉 This creates:

MySQL pods
MySQL services
Persistent storage

🔹 Step 3 — Verify MySQL is running

	kubectl get pods
	kubectl get svc

👉 You should see:

	my-release-mysql-primary → Running
	MySQL service → port 3306

🔹 Step 4 — deploy phpmyadmin with its configuration for Mysql DB access

	kubectl apply -f db-config.yaml
	kubectl apply -f db-secret.yaml
	kubectl apply -f phpmyadmin.yaml

🔹 Step 5 — access phpmyadmin and login to mysql db

	kubectl port-forward svc/phpmyadmin-service 8081:8081

Open in browser:

	http://localhost:8081
	
🔹 Step 9 — Login to MySQL

Get root password:

	MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace default my-release-mysql -o jsonpath="{.data.mysql-root-password}" | base64 -d)
	echo $MYSQL_ROOT_PASSWORD

Connect to Mysql:

	kubectl run my-release-mysql-client --rm --tty -i --restart='Never' --image docker.io/bitnamilegacy/mysql:latest --namespace default --env MYSQL_ROOT_PASSWORD=$MYSQL_ROOT_PASSWORD --command -- bash

Inside:

	mysql -h my-release-mysql-primary.default.svc.cluster.local -uroot -p"$MYSQL_ROOT_PASSWORD"

<img width="1457" height="586" alt="Screenshot 2026-04-01 at 11 39 48 AM" src="https://github.com/user-attachments/assets/6da16555-4341-4f47-9df9-287b2ecd3d68" />

  
🟢 EXERCISE 3: Deploy your Java application
You deploy your Java application using Fargate with 3 replicas using the same setup as before.

Setup Continuous Deployment with Jenkins

🔹 Create namespace my-app to deploy our java application, because we are deploying java-app with fargate profile. And fargate profile we 

	create applies for my-app namespace. 
	kubectl create namespace my-app

🔹 We now have to create all configuration and secrets for our java app in the my-app namespace

🔹 Create my-registry-key secret to pull image 
	
	DOCKER_REGISTRY_SERVER=docker.io
	DOCKER_USER=your dockerID, same as for `docker login`
	DOCKER_EMAIL=your dockerhub email, same as for `docker login`
	DOCKER_PASSWORD=your dockerhub pwd, same as for `docker login`

	kubectl create secret -n my-app docker-registry my-registry-key \
	--docker-server=$DOCKER_REGISTRY_SERVER \
	--docker-username=$DOCKER_USER \
	--docker-password=$DOCKER_PASSWORD \
	--docker-email=$DOCKER_EMAIL


🔹 Again from k8s-deployment folder, execute following commands. By adding the my-app namespace, these components will be created with Fargate profile

	kubectl apply -f db-secret.yaml -n my-app
	kubectl apply -f db-config.yaml -n my-app
	kubectl apply -f java-app.yaml -n my-app


🟢 EXERCISE 4: Automate deployment
Now your application is running, and when you or others make changes to it, Jenkins pipeline builds the new image, but you have to manually deploy it into the cluster. From your experience you know how annoying that is for you and your team, so you want to automate deploying to the cluster as well.

Setup automatic deployment to the cluster in the pipeline.


🟢 EXERCISE 5: Use ECR as Docker repository
Now your application is running, and when you or others make changes to it, Jenkins pipeline builds the new image, but you have to manually deploy it into the cluster. From your experience you know how annoying that is for you and your team, so you want to automate deploying to the cluster as well.

So, the company wants to use ECR instead, again to have everything on 1 platform and also to let AWS manage the repository including storage, cleanups etc. Therefore you:

Replace the docker repository in your pipeline with ECR

🟢 Exercise 4 & 5: Automate deployment & Use ECR as Docker repository 
 
Current cluster setup
At this point, you already have an EKS cluster, where:

✅ Mysql chart is deployed and phpmyadmin is running too
✅ my-app namespace was created
✅ db-config and db-secret were created in the my-app namespace for the java-app
✅ my-registry-key secret was created to fetch image from docker-hub
✅ your java app is also running

Steps to automate deployment for existing setup
🔹 Create an ECR registry for your java-app image

🔹 Locally, on your computer: Create a docker registry secret for ECR

	DOCKER_REGISTRY_SERVER=your ECR registry server - "your-aws-id.dkr.ecr.your-ecr-region.amazonaws.com"
	DOCKER_USER=your dockerID, same as for `docker login` - "AWS"
	DOCKER_PASSWORD=your dockerhub pwd, same as for `docker login` - get using: "aws ecr get-login-password --region {ecr-region}"
	
	kubectl create secret -n my-app docker-registry my-ecr-registry-key \
	--docker-server=$DOCKER_REGISTRY_SERVER \
	--docker-username=$DOCKER_USER \
	--docker-password=$DOCKER_PASSWORD

🔹 SSH into server where Jenkins container is running

	ssh -i {private-key-path} {user}@{public-ip}

🔹 Enter Jenkins container

	sudo docker exec -it {jenkins-container-id} -u 0 bash

🔹 Install aws-cli inside Jenkins container
- Link: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

	curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
	unzip awscliv2.zip
	./aws/install

🔹 Install kubectl inside Jenkins container
- Link: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

	apt-get update
	apt-get install -y apt-transport-https ca-certificates curl gnupg

	curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
	chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg

	echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | tee /etc/apt/sources.list.d/kubernetes.list

	chmod 644 /etc/apt/sources.list.d/kubernetes.list
	apt-get update
	apt-get install -y kubectl

🔹 Install envsubst tool
- Link: https://command-not-found.com/envsubst

	apt-get update
	apt-get install -y gettext-base

🔹 create 2 "secret-text" credentials for AWS access in Jenkins: 

	- "jenkins_aws_access_key_id" for AWS_ACCESS_KEY_ID 
	- "jenkins_aws_secret_access_key" for AWS_SECRET_ACCESS_KEY    

🔹 Create 4 "secret-text" credentials for db-secret.yaml:
	
	- id: "db_user", secret: "my-user"
	- id: "db_pass", secret: "my-pass"
	- id: "db_name", secret: "my-app-db"
	- id: "db_root_pass", secret: "secret-root-pass"

🔹 Set the correct values in Jenkins for following environment variables: 
	
	- ECR_REPO_URL
	- CLUSTER_REGION

🔹 Create Jenkins pipeline using the Jenkinsfile in this branch, in the root folder

Make sure the paths to the k8s manifest files in the "deploy" stage of the Jenkinsfile are all correct!!

🟢 EXERCISE 6: Configure Autoscaling
Now your application is running, whenever a change is made, it gets automatically deployed in the cluster etc. This is great, but you notice that most of the time the 3 nodes you have are underutilized, especially at the weekends, because your containers aren't using that many resources. However, your company is still paying full price for all of the servers.

You suggest to your manager that you will be able to save the company some infrastructure costs by configuring autoscaling. Your manager is happy with this suggestion and asks you to configure it. So you:

Go ahead and configure autoscaling to scale down to a minimum of 1 node when servers are underutilized and maximum of 3 nodes when in full use.



