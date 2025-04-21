## Kubernetes Autoscaling with KEDA and Karpeneter

KEDA (Kubernetes Event-Driven Autoscaler) extends native Kubernetes autoscaling capabilities by enabling container scaling based on event processing requirements rather than traditional resource metrics alone. This powerful solution allows DevOps teams to deploy scalable workloads that respond directly to application-level metrics and event queues.
As a Kubernetes-native solution, KEDA integrates seamlessly with existing components like the Horizontal Pod Autoscaler through custom resource definitions (CRDs). One of its distinctive features is the ability to implement time-based scaling patterns through cron schedules, providing precise control over when and how resources scale.
KEDA's versatility makes it equally effective in both cloud and edge environments, eliminating the need for external dependencies while maintaining complete compatibility with standard Kubernetes practices.
Complementing KEDA's capabilities, Karpenter optimizes infrastructure provisioning by automatically deploying the optimal compute resources exactly when needed. This intelligent node provisioning system ensures your Kubernetes cluster maintains perfect resource alignment with application demands.
Together, these technologies enable organizations to maximize cloud efficiency through rapid, precise compute provisioning that scales with your applications' actual requirements—creating a responsive, cost-effective Kubernetes infrastructure.RetryClaude can make mistakes. Please double-check responses.

## Autoscaling Kubernetes Workloads with KEDA and Karpenter: A Time-Based Approach

In this project, we demonstrate how to implement efficient autoscaling using KEDA (Kubernetes Event-driven Autoscaling) based on temporal patterns via cron schedules. The solution orchestrates pod scaling according to predefined time intervals specified in the KEDA configuration.
When KEDA initiates scaling operations, pods are created until they reach the node's resource capacity. Once the node is fully utilized, additional pods enter a pending unschedulable state. At this point, Karpenter automatically provisions new nodes to accommodate the pending workload, allowing the remaining pods to be scheduled and deployed successfully.
The following sections provide a detailed, step-by-step walkthrough of this process, illustrating how these complementary technologies work together to create a robust, time-based autoscaling solution.

![diagram-export-4-20-2025-9_58_15-PM](https://github.com/user-attachments/assets/c2147cd6-d67b-4e2d-af49-ef21ecb7889c)

### Karpenter Setup

 We will follow official Karpenter Documentation as given in the below link :

https://karpenter.sh/docs/getting-started/getting-started-with-karpenter/

Please follow Step 1 till Step 5 : This involves setting up the pre-requisites, Creating EKS Cluster, Installing Karpenter, setting up necessary IAM and RBAC policies and Configuring NodePool.

Once the Cluster is ready, we should see the below screen :

<img width="1273" alt="Screenshot 2025-04-18 at 4 45 21 PM" src="https://github.com/user-attachments/assets/e60bb651-0c33-4525-a451-0877cf752bb8" />


Setting up IAM service linked role. If the role already exists, we will see the below message as shown in the screenshot :

<img width="1372" alt="Screenshot 2025-04-18 at 4 45 30 PM" src="https://github.com/user-attachments/assets/3e590e75-09c8-4c76-8759-ed93ce6123b3" />

We can now install the Karpenter :

<img width="1263" alt="Screenshot 2025-04-18 at 4 45 43 PM" src="https://github.com/user-attachments/assets/f3aaab6d-7a92-4ba8-8416-fab42525d0a4" />

Next step would be to configure NodePool :

<img width="1071" alt="Screenshot 2025-04-18 at 4 46 00 PM" src="https://github.com/user-attachments/assets/a594049a-2735-4f19-98d7-7c20e9414a24" />

### KEDA Setup :

Step 1 : Adding KEDA Core Repo to Helm :

helm repo add kedacore https://kedacore.github.io/charts

Step 2: Helm Repo Update 

helm repo update

Step 3 : Install keda Helm chart

helm install keda kedacore/keda

This should install KEDA as shown in the below screenshot :

<img width="1540" alt="Screenshot 2025-04-18 at 4 46 13 PM" src="https://github.com/user-attachments/assets/928ea4c6-1798-4065-a574-8e5764accbb5" />

### Deploying the application yaml file 

For this project, we will use the deployment.yaml file present in the repository. It is the sample Spring boot application created in the below github repository 

https://github.com/ahuja012002/AWS-Springboot-EKS

Once the application is deployed using kubectl apply -f deployment.yaml 
We should be able to access the application using load balancer external IP :

<img width="1186" alt="Screenshot 2025-04-20 at 10 12 52 PM" src="https://github.com/user-attachments/assets/396adf5f-4b3f-4efa-b0a4-4cc3edb30d99" />

### Deploy KEDA configuration file :

To activate KEDA we will deploy the scaled-object.yml file in the repository :

This file will be deployed using kubectl apply -f scaled-object.yml

This file kicks off a Cron schedule for KEDA which starts at 25th min of every hour and ends at 50th min of every hour. We have followed America/New York Time zone.

Also, it has capped minimum replica to 0 and maximum replica to 50. The desired replica can be changed as per our requirement. I have set it up as 50 initially to show scale out and then back to 4 to show scale in.

Based on the schedule, KEDA creates new pods upto 50 in number and this also triggers Karpenter to create new node as 50 pods cannot be accomodated into existing nodes. As shown in the below screenshot :

<img width="1201" alt="Screenshot 2025-04-18 at 4 29 52 PM" src="https://github.com/user-attachments/assets/029cb153-1ccb-4661-81ed-70fc20f94050" />


<img width="1777" alt="Screenshot 2025-04-18 at 4 29 37 PM" src="https://github.com/user-attachments/assets/50233c0e-0510-4022-8d7d-76dbf09ae169" />

<img width="1489" alt="Screenshot 2025-04-18 at 4 29 17 PM" src="https://github.com/user-attachments/assets/efd6ef93-6ca4-4a7d-9247-2cdaa468ea24" />

The highlighted row in the above screenshot shows the node created by Karpenter.

Once the time passed based on the cron schedule and scale in event kicks in and the additional node is terminated by Karpenter as shown below :

<img width="1367" alt="Screenshot 2025-04-18 at 4 43 49 PM" src="https://github.com/user-attachments/assets/2f3b43a7-f8f9-4fe2-8dca-95c1ba0b67ad" />

### This shows how KEDA and Karpenter can work together to provide Event Driven Auto scaling in a Kubernetes Cluster.



