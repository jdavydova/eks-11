### These commands are used in the Deploy to EKS from Jenkins lecture

##### Install kubectl on Jenkins server
 
```sh
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl; chmod +x ./kubectl; mv ./kubectl /usr/local/bin/kubectl
```

##### Install aws-iam-authenticator on Jenkins server

 ```sh
 curl -Lo aws-iam-authenticator https://github.com/kubernetes-sigs/aws-iam-authenticator/releases/download/v0.6.11/aws-iam-authenticator_0.6.11_linux_amd64
 chmod +x ./aws-iam-authenticator
 mv ./aws-iam-authenticator /usr/local/bin
```

##### Copy config file to Jenkins server

 ```sh
 docker cp config "YOUR DOCKER CONTAINER ID":/var/jenkins_home/.kube/
 ```
