## EKS Learning

### [Amazon EKS Primer](https://explore.skillbuilder.aws/learn/course/57)  
AWS beginners course about EKS fundamentals.  
Cover the following topics: Architecture, Installation, Basic networking, Basic storage, and Upgrade procedure.  

#### Cluster installation (using eksctl):
eksctl uses CloudFormation to create the entire eks stack which includes: EKS, IAM roles, NodeGroup, VPC, NAT (if needed), etc.  
It also helps to handle day 2 tasks such as configuration changes or cluster upgrades.  
1. Create the cluster (using the YAML file in this path):  
`eksctl create cluster -f clusterconfig.yaml`
2. Get the cluster's `kubeconfig` file:
   `aws eks update-kubeconfig --name eks-training`

#### Cluster upgrade:
1. Control plane upgrade:
   Performed completely by AWS by creating a duplicate upgraded version control plane and migrating to it (green-blue).
   `eksctl upgrade cluster --name=<your-cluster-name> --version=<new-k8s-version> --approve`
2. Node Group upgrade:
   In the case of a managed node group, the upgrade is performed in a canary way.
   `eksctl upgrade nodegroup --cluster=<your-cluster-name> --name=<nodegroup-name>`

**Notes:**
- The control plane is completely hidden.
- AWS uses service accounts to connect between IAM and the cluster itself.
- OIDC service should be configured (other than IAM) to allow user authentication.
- AWS uses CSI and StorageClasses to provide PVCs. It supports both block and file storage via EBS and EFS accordingly. (require CSI driver installation and a dedicated IAM role).
- Kubernetes Ingress and service type LoadBalancer objects are implemented by ELB while ingress uses layer 7 and LoadBalancer uses layer 4 LB. (require controller installation and a dedicated IAM role).

