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
- Create k8s manifest files for all the resoures/Helm package and test it in DEV k8s cluster/local Minikube cluster
- Create Kubernetes manifest files/ Helm chart for each service ( Deployment, service, config map, secrets, ingress, etc)
- For Redis and MongoDB, let's go for managed services, which is the best option.

### Create and manage k8s cluster across multiple regions

- Create two EKS clusters in two different regions, one as a primary cluster and the other as a failover cluster.
- Choose replication approach. Idea is to run exact copy of all the services in failover cluster.
- For logging/monitoring/alerting we can use cloud-specific services (cloud watch, container insights, cloud watch alerts, etc) as we are running k8s cluster in the same cloud

#### Architeture Diagram

![Alt Multi Region Kubernetes Cluster](https://raw.githubusercontent.com/akilans/havels-task/main/images/EKS-multi-region.png?raw=true)

#### Services

- EKS cluster - spin 2 eks cluster in 2 different regions
- ECR - To store docker images
- EC2 - Kubernetes worker nodes
- ALB - Load balancer provisioned by ingress controller to direct traffic to correct services
- AWS Load Balancer Controller - Ingress controller for EKS cluster
- IAM, Roles, User, K8s RBAC, service accounts for accessing EKS cluster
- AWS cloudwatch, Fluentbit, Cloudwatch agent, Container Insights, SNS for logging, Monitoring and alerting
- Gitops- To deploy microservices into kubernetes cluster
- Amazon ElastiCache for Redis - Global Datastore - Cross region redis cluster
- Amazon DocumentDB Global Clusters - Cross region mongodb cluster
- Gitops - Argocd or Flux to deploy kubernetes resources to EKS cluster by CI server(jenkins, AWS Code pipeline)
- AWS global accelerator - Networking service to route traffic to applications deployed in 2 different regions

### Create and manage k8s cluster across multiple cloud providers

- Create two EKS clusters in two different cloud providers, one as a primary cluster and the other as a failover cluster.
- Choose replication approach. Idea is to run exact copy of all the services in failover cluster.
- For logging/monitoring/alerting we can use these services (Prometheus, Elasticsearch, Grafana etc) as we are running k8s cluster in different cloud providers

#### Architeture Diagram

![Alt Multi Cloud Kubernetes Cluster](https://raw.githubusercontent.com/akilans/havels-task/main/images/Multi-Cloud-K8S.png?raw=true)

#### Services

- EKS and AKS cluster - spin eks cluster in AWS cloud and AKS cluster in Azure
- ECR/ACR - To store docker images
- EC2 and Azure VMs - Kubernetes worker nodes
- ALB/ - Load balancer provisioned by ingress controller to direct traffic to correct services
- AWS Load Balancer Controller - Ingress controller for EKS cluster
- Nginx Ingress contoller for AKS
- IAM, Roles, User, K8s RBAC, service accounts for accessing EKS/AKS cluster
- Prometheus, Elasticsearch, Grafana for logging, Monitoring and alerting
- Gitops- To deploy microservices into kubernetes cluster
- Amazon ElastiCache for Redis - Global Datastore - Cross region redis cluster
- Amazon DocumentDB Global Clusters - Cross region mongodb cluster
- Gitops - Argocd or Flux to deploy kubernetes resources to EKS and AKS cluster by CI server(jenkins, AWS Code pipeline)
- Global Load balancer - To route traffic between applications deployed on 2 different cloud providers
