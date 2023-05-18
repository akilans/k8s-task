# Design HA - Kubernetes Cluster

- You have a SaaS cloud for IoT that is collection of microservices(say a few dozens) that are deployed on virtual machines or Azure app service plan/VMs or equivalent.
- These microservices rely on mongo and redis for data storage. Assume relevant infra services such as load balancer, logging etc as appropriate (call it out) to make deployment complete.
- You have to migrate these microservices from VM/equivanent to kubernetes platform.
- How will you plan and execute this transformation exercise?.
- How will you make services available on kubernetes available across multiple regions of cloud provider?.
- How will you make microservices available on hybrid cloud env – say AWS and Azure?.
- Assess impact on data layer and cover that too when you analyze above problems.
- Pls call out important decisions, actions, risks, outcome.

### Migration Plan and Execution

- Dockerize all the applications and test them by running them as containers
- If possible test everything with docker-compose locally
- Create images for all the applications and push them to the container registry(ECR)
- Create k8s manifest files for all the resources/Helm package and test it in DEV k8s cluster/local Minikube cluster
- Create Kubernetes manifest files/ Helm chart for each service ( Deployment, service, config map, secrets, ingress, etc)
- For Redis and MongoDB, let's go for managed services, which is the best option.
- Plan for network configuration to share data between applications running on two k8s clusters

### Challenges

- Choosing a multi-region K8s cluster. Replication approach, or run a single K8s cluster and spread workload across multiple regions.
- Managed Kubernetes [EKS] works at the region level. So we will go with the replication approach
- We need a global load balancer to direct traffic between services deployed on two different EKS clusters. AWS global accelerator is the best option here.
- We need to go for global data store option for both MongoDB and Redis
- Need to connect and configure networking settings to share data between applications running on two different cluster
- Deploy applications with the same configuration with two different K8s clusters. Gitops is the best option(ArgoCD)

### Create and manage k8s cluster across multiple regions

- Create two EKS clusters in two different regions, one as a primary cluster and the other as a failover cluster.
- Choose a replication approach. The idea is to run an exact copy of all the services in the failover cluster.
- For logging/monitoring/alerting we can use cloud-specific services (cloud watch, container insights, cloud watch alerts, etc) as we are running a k8s cluster in the same cloud

#### Architeture Diagram

![Alt Multi Region Kubernetes Cluster](https://raw.githubusercontent.com/akilans/havels-task/main/images/EKS-multi-region.png?raw=true)

#### Services

- EKS cluster - spin two EKS clusters in two different regions
- ECR - To store docker images
- EC2 - Kubernetes worker nodes
- ALB - Load balancer provisioned by ingress controller to direct traffic to correct services
- AWS Load Balancer Controller - Ingress controller for EKS cluster
- IAM, Roles, User, K8s RBAC, service accounts for accessing the EKS cluster
- AWS cloud watch, Fluentbit, Cloudwatch agent, Container Insights, SNS for logging, Monitoring, and alerting
- Gitops- To deploy microservices into the Kubernetes cluster
- Amazon ElastiCache for Redis - Global Datastore - Cross region Redis cluster
- Amazon DocumentDB Global Clusters - Cross region Mongodb cluster
- Gitops - Argocd or Flux to deploy Kubernetes resources to EKS cluster by CI server(Jenkins, AWS Code pipeline)
- AWS global accelerator - Networking service to route traffic to applications deployed in 2 different regions

### Create and manage k8s cluster across multiple cloud providers

- Create two K8s clusters in two different cloud providers, one as a primary cluster and the other as a failover cluster.
- Choose the replication approach. The idea is to run an exact copy of all the services in the failover cluster.
- For logging/monitoring/alerting we can use these services (Prometheus, Elasticsearch, Grafana, etc) as we are running k8s cluster in different cloud providers

#### Architeture Diagram

![Alt Multi Cloud Kubernetes Cluster](https://raw.githubusercontent.com/akilans/havels-task/main/images/Multi-Cloud-K8S.png?raw=true)

#### Services

- EKS and AKS cluster - spin EKS cluster in AWS cloud and AKS cluster in Azure
- ECR/ACR - To store docker images
- EC2 and Azure VMs - Kubernetes worker nodes
- ALB/ - Load balancer provisioned by ingress controller to direct traffic to correct services
- AWS Load Balancer Controller - Ingress controller for EKS cluster
- Nginx Ingress controller for AKS
- IAM, Roles, User, K8s RBAC, service accounts for accessing EKS/AKS cluster
- Prometheus, Elasticsearch, and Grafana for logging, Monitoring, and alerting
- Gitops- To deploy microservices into the Kubernetes cluster
- Amazon ElastiCache for Redis - Global Datastore - Cross region Redis cluster
- Amazon DocumentDB Global Clusters - Cross region Mongodb cluster
- Gitops - Argocd or Flux to deploy Kubernetes resources to EKS and AKS cluster by CI server(Jenkins, AWS Code pipeline)
- Global Load balancer - To route traffic between applications deployed on 2 different cloud providers
