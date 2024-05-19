# Three-Tier Web Application Deployment on AWS EKS using AWS EKS, ArgoCD, Prometheus, Grafana, and Jenkins
Welcome to the Three-Tier Web Application Deployment project! 🚀

Step by step: 


1. IAM user setup -> creadential

2. Iac -> create EC2 run jenkins

3. Jenkins server -> config jenkins, docker, sonarQ, Terraform, Kubectl, aws cli, 
	trivy, stream deploy
	
4. EKS -> Jenkins -> EKS cluster managed by K8s on AWS, run application

5. LB -> ALB for cluster, -high avaibility

6.ECR -> same Docker registery, repo store docker images

7. ArgoCD -> for CD & GitOps, make simple deployment process

8. SonarQ Integration: analysis DevSecOps, make sure code integrity throughout the 
	development lifecycle

9. Jenkins Pipelines: -> created streamline -> deploy be, fe to eks

10.Monitoring: implement monitoring for eks cluster using helm, prometheus, grafana, enabling real-time 
	insights into system performance and health
	
11. ArgoCD Application Deployment: deploy 3-tier app -> (db,be,dr,ingress component)

12. DNS: custom sub domain -> enabling convenient access to app

13. Data Persistence: ensure data persistence

14.  Monitoring: EKS cluster performance using Grafana 
		ensure ongoing optimization and efficiency
		
		
Step by step complete CICD lab:

1. Create credential for user -> add credential to Terraform using command line:
	`aws configure` -> add credential/region/json/
		-	region: ap-south-1
		-	output format: json 

2. Install dependency -> terraform, awscli to jenkins server (ec2)
	- make jenkins server run linux/version 22.04/amd64 (ami-05e00961530ae1b55 (64-bit (x86)))
	- t2.medium / ssd 30GB / jenkins-key /
	- open port [22, 8080, 9000, 9090, 80] for sg
	- add userdata "install_tools.sh"
	after all, check version dependencies
			jenkins --version
			docker --version
			docker ps
			terraform --version
			kubectl version
			aws --version
			trivy --version
			eksctl version
			
3. Go to config Jenkins Server
	- go to <IP_public>:8080 
	- password: sudo cat /var/lib/jenkins/secrets/initialAdminPassword

4. 	Deploy EKS cluster by using eksctl 
	1. ```aws configure``` on jenkins server
	2. add plugin -> available plugin -> install "aws credential" / "aws step"
	3. login again
	4. add credentials -> Dashboard
						Manage Jenkins
						Credentials
						System
						Global credentials (unrestricted)
	5.  add username and password GITHUB
	
5. Create EKS cluster 
    Run file `Infrastructure-TF` layer2 to create EKS Cluster with 2 nodes
	# Update kubeconfig to use the newly created EKS cluster
	aws eks update-kubeconfig --region ap-south-1 --name "Three-Tier-K8s-EKS-Cluster"
	*********** delete resources:::   eksctl delete cluster --region=ap-northeast-1 --name=k8s-demo :::eteled*******************
	
6. Config LB on EKS ( APP need INGRESS controller )
	
	+ Download binary:
		curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
	+ Create IAM policy:
		aws iam create-policy - policy-name AWSLoadBalancerControllerIAMPolicy - policy-document file://iam_policy.json
	+ Create OIDC provider:
		eksctl utils associate-iam-oidc-provider --region=ap-south-1 --cluster=Three-Tier-K8s-EKS-Cluster --approve
	+ Create Svc account:
		eksctl create iamserviceaccount --cluster=Three-Tier-K8s-EKS-Cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::179623033511:policy/AWSLoadBalancerControllerIAMPolicy --approve --region=ap-south-1
	+ run command deploy LB controller:
		sudo snap install helm --classic
		helm repo add eks https://aws.github.io/eks-charts
		helm repo update eks
		helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system - set clusterName=my-cluster - set serviceAccount.create=false - set serviceAccount.name=aws-load-balancer-controller
	+ verify:
		`kubectl get deployment -n kube-system aws-load-balancer-controller`
	
	AWS reference: https://docs.aws.amazon.com/eks/latest/userguide/lbc-helm.html#lbc-helm-install
	
	`$ aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json`
	`$ eksctl utils associate-iam-oidc-provider --region=ap-south-1 --cluster=Three-Tier-K8s-EKS-Cluster --approve`
	
	`$ eksctl create iamserviceaccount --cluster=Three-Tier-K8s-EKS-Cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::179623033511:policy/AWSLoadBalancerControllerIAMPolicy --approve --region=ap-south-1`
	
	```$ helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=my-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller```

	```$ kubectl create secret generic ecr-registry-secret --from-file=.dockerconfigjson=${HOME}/.docker/config.json --type=kubernetes.io/dockerconfigjson --namespace three-tier```

    `$ kubectl get secrets -n three-tier `