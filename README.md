# Overview

  The snaptrude NodeJS app is a sample setup to demonstrate a sample Express application with sockets. This code is packaged and run as a container on Kubernetes (EKS) on AWS Cloud. 

## Architecture and Traffic Flow

![Snaptrude App Traffic Flow](https://github.com/suhasagasthya/snaptrude-assignment/blob/main/snaptrude_sample_flow.jpg?raw=true)


## Deployment Steps and Explanation

1. Container Image for the nodeJS app
  - The first step involes packaging the code base into a container image which includes all the code dependencies, packages and also following the security best practises based into the container image. 
  - Two docker files are used for the same -> Dockerfile-base and Dockerfile.
  - the Dockerfile-base is a template base image for any nodeJS app which includes the security best practises such as running the app as non root user (uid=1001 and gid=2001), declraing the app name, data paths to be used in the image and running the same over a tiny busybox image. 
  - The Dockerfile uses the base image to just run the index.js and expose the container port
  - Both these images are built locally and pushed to a private dockerhub container registry

2. Platform to run the Container Image
  - The next step would be to have a platform to run these container images and allow user traffic
  - The platform used here is [Amazon Elastic Kubernetes Service](https://aws.amazon.com/eks/) on AWS. 
  - The EKS cluster is setup with 1 Node Group on spot instances in 1 Availability Zone only.
  - Components used in EKS:
    - Pod Network (CNI) -> [Calico](https://www.tigera.io/tigera-products/calico/)
    - Worker Nodes autoscaling -> [Cluster Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler)
    - Ingress Traffic Management -> [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/)
  - All the EKS components used are setup either via Helm charts, eks managed add-ons or manually for the purpose of the demo only.

3. Traffic Flow
  - DNS records are managed on [Cloudflare](https://www.cloudflare.com/en-gb/learning/what-is-cloudflare/) and the entire traffic is proxied over cloudflare network.
  - Any rate limits, WAF, SSL, Caching related configurations can be done at the edge through cloudflare configurations
  - Cloudflare tunnel is an ipsec tunnel from the deployment to cloudflare edge allowing exposing any private endpoint over the public internet Cloudflare Tunnel.


## Alternate Architectures/Approaches

Below are the alternate approaches or services that can be used to handle the deployment or traffic of the app

1. Traffic Management
  - AWS Load Balancers and using an Ingress Controller to route traffic to EKS
  - Using Route 53 for DNS records

2. Container Platform 
  - Amazon Elastic Container Service
  - Self Managed vanilla Kubernetes Cluster or Openshift
  - EKS with Fargate

3. Serverless Approach
  - Running the app on Lambda and using API Gateway to route traffic, dns managed via Route 53
  - AWS App Runner

4. Non Kubernetes Approach
  - The code can be deployed on ec2 instances managed via Auto Scaling Group (ASG) and traffic routed through Application Load Balancer to ASG
