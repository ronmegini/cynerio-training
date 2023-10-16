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

#### Ingress controller ([docs](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html)):
1. Get the IAM role:  
   `curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json`
3. Create IAM policy:  
   `aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json`
4. Create IAM role:  
   `eksctl create iamserviceaccount --cluster=my-cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::111122223333:policy/AWSLoadBalancerControllerIAMPolicy --approve`
5. Install using helm:  
   `helm repo add eks https://aws.github.io/eks-charts`  
   `helm repo update eks`  
   `helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=my-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller`  

**Notes:**
- The control plane is completely hidden.
- AWS uses service accounts to connect between IAM and the cluster itself.
- OIDC service should be configured (other than IAM) to allow user authentication.
- AWS uses CSI and StorageClasses to provide PVCs. It supports both block and file storage via EBS and EFS accordingly. (require CSI driver installation and a dedicated IAM role).
- Kubernetes Ingress and service type LoadBalancer objects are implemented by ELB while ingress uses layer 7 and LoadBalancer uses layer 4 LB. (require controller installation and a dedicated IAM role).

### [Running Containers on Amazon EKS & Labs](https://explore.skillbuilder.aws/learn/course/15132/running-containers-on-amazon-eks-hebrew)  

#### Ingress vs Service type LoadBalancer
- Both of them are used for exposing kubernetes workload externally.
- The LoadBalancer is handled by a controller that creates an `AWS Elastic Load Balancer` which handles TCP level communication.
- The Ingress is handled by a controller that creates `AWS Application Load Balancers` which handle HTTP/HTTPS level communication with advanced 7-layer features.
- LoadBalacer controller is ready out of the box with eks cluster, ingress controller needed to be installed separately.

#### [Monitoring](https://explore.skillbuilder.aws/learn/course/15132/play/72947/collecting-metrics)
- Many tools allow monitoring, logging, and observability for EKS. such as Prometheus Monitoring Stack (managed by AWS and open-source), ELK, Splunk, CloudWatch
- Whatever tool is chosen, a collector is required. (such as fluentd, node exporter, etc.)
- CloudWatch support both Metrics and logs ([installation guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/deploy-container-insights-EKS.html))

#### Auto Scaler
- There several types of auto scalers: node level (cluster auto scaler), replicas level (HPA), resources level (VPA)
##### [Cluster Auto Scaler](https://artifacthub.io/packages/helm/cluster-autoscaler/cluster-autoscaler)
- Installation with an add-on, helm chart, or manual installation.
- Scale up and down node groups
- Add nodes when needed, remove nodes when unneeded, and reschedule pods for better node utilization
- Allow custom settings. Example:
  ```
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: cluster-autoscaler
    namespace: kube-system
  data:
    config.json: |
      {
        "auto-discovery": {
          "asg-tag": "k8s.io/cluster-autoscaler/enabled",
          "enable-default-aws-asg-tags": false
        },
        "balance-similar-node-groups": true,
        "skip-nodes-with-system-pods": false,
        "skip-nodes-with-local-storage": false,
        "expander": "least-waste",
        "scaling-policy": "Auto"
      }
   ```
##### Horizontal Pod Autoscaler
- Installation with a helm chart or manual installation.
- A metrics server is needed.
- Scale up/down the replicas of each managed deployment based on the resource utilization out of the configured limits.
- Allow custom settings. Example:
   ```
   apiVersion: autoscaling/v2beta2
   kind: HorizontalPodAutoscaler
   metadata:
     name: my-app-hpa
   spec:
     scaleTargetRef:
       apiVersion: apps/v1
       kind: Deployment
       name: my-app-deployment
     minReplicas: 2
     maxReplicas: 10
     metrics:
       - type: Resource
         resource:
           name: cpu
           targetAverageUtilization: 50
   ```
##### Vertical Pod Autoscaler
- Installation with a helm chart or manual installation.
- A metrics server is needed.
- Scale up/down the resources of each managed deployment based on the resource utilization out of the configured limits.
- Allow custom settings. Example:
```
   apiVersion: autoscaling.k8s.io/v1
   kind: VerticalPodAutoscaler
   metadata:
     name: my-app-vpa
   spec:
     targetRef:
       apiVersion: "apps/v1"
       kind:       "Deployment"
       name:       "my-app-deployment"
     updatePolicy:
       updateMode: "On"
     resourcePolicy:
       containerPolicies:
         - containerName: "my-app-container"
           minAllowed:
             memory: "64Mi"
             cpu: "100m"
           maxAllowed:
             memory: "512Mi"
             cpu: "1000m"
```

#### Networking
- The EKS lives inside of a VPC, which means that the pods' IPs are VPC addresses.
- The egress traffic made via a NAT and Internet Gateway (in case of a private subnet)
- The ingress traffic made via a LoadBalancer or Ingress.
- Route table shell be configured for egress network via the Internet Gateway.
- Network rules should be configured - By security group for the VMs, and by NACL for the entire subnet.
- DNS service with AWS Route 53. The records inside the VPC are configured with a private hosted zone, while the public records are configured in the public record zone.
- Multiple subnets can be attached to EKS (for example in case addresses are over)

#### Authorization & Authentication
- Authentication implemented with AWS IAM, Authorization implemented with EKS RBAC.
- IAM Group=RBAC Group; IAM User=RBAC User; IAM Role=RBAC Role Binding; IAM Policy=RBAC Role
- To map between IAM users and roles to RBAC permissions aws-auth config map is used. Example:
  ```
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: aws-auth
    namespace: kube-system
  data:
    mapRoles: |
      - rolearn: <ARN of NodeInstanceRole>
        username: system:node:{{EC2PrivateDNSName}}
        groups:
          - system:bootstrappers
          - system:nodes
    mapUsers: |
      - userarn: arn:aws:iam::<account-id>:user/<username>
        username: <eks-username>
        groups:
          - <k8s-group>
  ```
  or  
  `eksctl create iamidentitymapping --cluster <name> --arn <arn> --group <Kubernetes RBAC group to which the specified AWS IAM role or user should be mapped> --username <Kubernetes username that the role mapped to>`  
- To grant pods permissions to access AWS services, we need to map Kubernetes ServiceAccount to AWS role.
- To map between SA and AWS role we need to configure OIDC provider. Example:  
  `eksctl utils associate-iam-oids-provider --cluster <name> --approve --region <region>`
- Create SA with a coresponding IAM role (allow s3 read access):  
  `eksctl create iamserviceaccount --name aws-s3-read --namespace default --cluster prod --attach-policy-arn arn:aws:iam:: aws:policy/AmazonS3ReadonlyAccess --approve --region us-west-2`

**Notes:**
- Similar to `oc get co`, kubernetes expose `kubectl get cs` to tget the control plane compenents statuses.
- The connection betweeen IAM identity to kubernetes RBAC configured by ConfigMap named aws-auth which map between aws role to rbac group.
- Flux is another kind of gitOps tool, similar to ArgoCD
