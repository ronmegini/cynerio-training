### EKS Learining

[Amazon EKS Primer](https://explore.skillbuilder.aws/learn/course/57)  
AWS begginers course about EKS fundamentels.  
Cover the following topics: Architecture, Installation, Basic networking, Basic storage and Upgrade procedure.  

Cluster installation (using eksctl):
eksctl use CloudFormation to create the entire eks stack which includes: EKS, IAM roles, NodeGroup, VPC, NAT (if needed), etc.  
It also help to handle day 2 tasks such as configuration change or cluster upgrade.  
Create the cluster (using the yaml file in this path):  
`eksctl create cluster -f clusterconfig.yaml`

