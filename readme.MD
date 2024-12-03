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

