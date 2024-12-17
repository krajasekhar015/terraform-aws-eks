# Terraform-AWS-EKS

we use terraform to create cluster and upgrade 

1. we will create cluster 
2. we will run application 
3. we will upgrade cluster 

- We have to make sure application should be running continuously even at the time of cluster upgrade (For safety, we will announce down-time)
- This is an admin activity 

![alt text](img/terraform-aws-eks.svg)

-> In AWS account and we should create in our expense VPC and we have three subnets - private, public and database
-> EKS cluster and nodes are always in private subnet 
-> we can have any number of nodes -> this is node group 
-> Since EKS cluster and managenodegroup are private and are internal, we can allow all traffic from both sides
    - We can allow all traffic from control plane to node group
    - We can allow all traffic from nodegroup to EKS cluster (Control Plane)
-> traffic comes from route53 -> to load balancer -> to worker nodes -> to RDS (i.e database)
-> Bastion host can be in public subnet
-> Here loadbalancer is going to be ingress controller 


**00-VPC**
- Previoulsy we have created expense VPC in `expense-terraform-dev`. There is no changes in it.

```
cd terraform-aws-eks/00-vpc
```
```
terraform init 
```
```
terraform plan 
```
```
terraform apply -auto-approve
```

**10-SG**

- Here, we will create security groups for Ingress_ALB, Bastion, EKS Control Plane, NodeGroup, Mysql
- Now, we need to decide traffic
    - Ingress_ALB will get traffic on port.no:443 from internet
    - Workernodes will get traffic on nodeport.no: from ingressALB
        - Here, we need to know the range of nodeport `30000 - 32767`
    - Nodegroup receiving traffic from EKS contol plane on all ports
        - from_port and to_port should be 0 and protocol should be given as -1
    - EKS control plane receives traffic from nodegroup on all ports
        - from_port and to_port should be 0 and protocol should be given as -1
    - Suppose pod-1 is in node-1 and pod-2 is in node-2, then
        - node2 should allow traffic from node-1 through CIDR 10.0.0.0/16. Because workernodes are temporary (based on traffic they will be created or deleted) we cannot give IP address specifically.
        - Node receiving traffic from VPC CIDR on all ports. We should give VPC CIDR (10.0.0.0/16)
    - Node is accepting traffic from bastion on port.no:22
    - eks control plane should allow traffic from bastion host on port.no:443
    
**20-Bastion**

- Here, in bastion.sh we will install all the kubernetes clients
- Using bastion, we will connect to cluster

**40-EKS**

- Here, we are using open-source EKS module which is official module
- After creating cluster, login to bastion and do `aws configure`
- Then apply the following command:
```
aws eks update-kubeconfig --region us-east-1 --name expense-dev
```
```
kubectl get nodes
```

