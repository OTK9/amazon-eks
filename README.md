# amazon-eks
Creating a cluster using EKSCTL
https://devopscube.com/create-aws-eks-cluster-eksctl/	
vi eks-cluster.yaml

apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: eks-spot-cluster
  region: us-east-1

vpc:
  id: "vpc-0872ded5f6ef1ae82"
  cidr: "172.31.0.0/16"
  subnets:
    public:
      us-east-1a: { id: subnet-03d1fb0fb5819c675 }
      us-east-1b: { id: subnet-092720cb27e360ff5 }
      us-east-1c: { id: subnet-0ca177c0e31967383 }

managedNodeGroups:
  - name: ng-db
    instanceType: t3.small
    labels: { role: builders }
    minSize: 2
    maxSize: 4
    ssh: 
      allow: true
      publicKeyName: devops
    tags:
      Name: ng-db
  - name: ng-spot
    instanceType: t3.medium
    labels: { role: builders }
    minSize: 3
    maxSize: 6
    spot: true
    ssh: 
      allow: true
      publicKeyName: devops
    tags:
      Name: ng-spot
	  
eksctl create cluster -f eks-cluster.yaml

aws eks update-kubeconfig --region us-east-1 --name eks-spot-cluster

# installing metric server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# VELERO Backup:

# Prerequisites
# eksctl v0.155.0 or greater
# cluster must be configured with an EKS IAM OIDC Provider. This is a requirement for IAM roles for service account.
# Each cluster must have the Amazon EBS CSI driver installed.
# AWS CLI version 2
# Helm v3
# kubectl

# Create an IAM OIDC provider for your cluster
# Determine the OIDC issuer ID for your cluster.
cluster_name=eks-spot-cluster
oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)
echo $oidc_id
# Determine whether an IAM OIDC provider with your cluster's issuer ID is already in your account. If output is returned, then you already have an IAM OIDC provider for your cluster and you can skip the step after this. If no output is returned, then you must create an IAM OIDC provider for your cluster.
aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4
# Create an IAM OIDC identity provider for your cluster with the following command.
eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve

# Creating the Amazon EBS CSI driver IAM role
# To create your Amazon EBS CSI plugin IAM role with eksctl
eksctl create iamserviceaccount \
    --name ebs-csi-controller-sa \
    --namespace kube-system \
    --cluster eks-spot-cluster \
    --role-name AmazonEKS_EBS_CSI_DriverRole \
    --role-only \
    --attach-policy-arn arn:aws-us-go:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
    --approve

# Adding the Amazon EBS CSI driver add-on to your cluster
eksctl create addon --name aws-ebs-csi-driver --cluster eks-spot-cluster --service-account-role-arn arn:aws:iam::891377401443:role/AmazonEKS_EBS_CSI_DriverRole --force
