##### EKSCTL create cluster command

```
eksctl create cluster \
--name demo-cluster \
--version 1.35 \
--region eu-north-1 \
--nodegroup-name demo-nodes \
--node-type t3.micro \
--nodes 2 \
--nodes-min 1 \
--nodes-max 3
```
