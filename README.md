# Udacity Cloud DevOps Nanodegree Capstone Project
The challenge was: 
1. Building CI/CD Pipeline with Jenkins
2. Building Docker containers in pipelines and use ECR to store image
3. Working with Ansible and CloudFormation to deploy clusters
4. Building Kubernetes clusters using AWS EKS

## Learning Objectives
1. Spinning up & Creating Jenkins
2. Configuring Jenkins with the needed plugins
3. Pushing Docker images to an private ECR repository
4. Pull Docker images from the private ECR repo
5. Deploy those images through Kubernetes

## Contents
The project have a structure as below:

```bash
├── Dockerfile
├── Jenkinsfile
├── README.md
├── urls.txt
├── aws
│   ├── aws-auth-cm.yaml
│   ├── cluster.yml
│   ├── cluster-params.json
│   ├── cluster-workers.yml
│   ├── cluster-workers-params.json
│   ├── container-registry.yml
│   ├── container-registry-params.json
│   ├── create_stack.sh
│   ├── network.yml
│   ├── network-params.json
│   ├── update_stack.sh
├── kubernetes
│   ├── deployment.yml
│   ├── service.yml
├── screenshots
│   ├── deployed_website.PNG
│   ├── kubectl_svc_pods.PNG
│   ├── lint_successfull.PNG
│   ├── Linting_Failure_01.PNG
│   ├── successful_pipeline.PNG
│   ├── worker_node.PNG

```
## How to Deploy the Infrastructure
* To begin with the CI/CD Pipeline, you need to have a EC2 Instance with Jenkins, aws-cli, Docker & kubectl deployed.
### Steps
1. Create repository on AWS ECR through
```
./create_stack UdacityCapStone-ECR container-registry.yml container-registry-params.json
```
2. Create the underlying networking (VPC, 3 public Subnets and the ControlPlane) through 
```
./create_stack.sh UdacityCapStone-Infrastructure network.yml network-params.json
```
3.  After completing the network creation step, deploy the K8S cluster (EKS) through
```
./create_stack.sh UdacityCapStone-K8S cluster.yml cluster-params.json
```
(This may take up to 15 mins.)

4. To enable nodes in the cluster, where the pods get deployed through Kubernetes  through 
```
./create_stack.sh UdacityCapStone-K8S-Workers cluster-workers.yml cluster-workers-params.json
``` 
5. The nodes only join, if they get the correct permissions assigned. This has to be done within the K8S Cluster through a ConfigMap
6. Run 
```
kubectl apply -f aws-auth-cm.yaml
```
7. Make sure to have your kubectl already configured with the EKS 
```
aws eks --region region-code update-kubeconfig --name cluster_name
```
8. To make sure everything was successfull `run kubectl get svc` which should show the kubernetes services.