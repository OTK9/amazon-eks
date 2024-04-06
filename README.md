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
