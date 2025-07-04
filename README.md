<<<<<<< HEAD
# Deploy a E-Commerce 3-Tier-App on AWS EKS

### Pre-requisites
- AWS CLI
- kubectl
- eksctl
- Helm

### Clone this project
```
git clone https://github.com/jocasantos/3-tier-app-eks.git
```

### Create a EKS cluster
```
eksctl create cluster --name demo-cluster-three-tier-1 --region eu-north-1
```

### Configure IAM OIDC provider 
```
export cluster_name=demo-cluster-three-tier-1
```
```
oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5) 
```
- Check if there is an IAM OIDC provider configured already

```
aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4
```
- If not, run the below command
```
eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
```
- Configure kubectl with the EKS Cluster
```
aws eks --region eu-north-1 update-kubeconfig --name $cluster_name
```

### ALB Configuration
- Download IAM policy
```
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json
```
- Create IAM Policy
```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```
- Create IAM Role
```
eksctl create iamserviceaccount \
  --cluster=<your-cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

### Deploy ALB controller
- Add helm repo
```
helm repo add eks https://aws.github.io/eks-charts
```
- Update the repo
```
helm repo update eks
```
- Install
```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \            
  -n kube-system \
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<region> \
  --set vpcId=<your-vpc-id>
```
- Verify that the deployments are running.
```
kubectl get deployment -n kube-system aws-load-balancer-controller
```

### EBS CSI Plugin configuration
- The Amazon EBS CSI plugin requires IAM permissions to make calls to AWS APIs on your behalf.
```
eksctl create iamserviceaccount \
    --name ebs-csi-controller-sa \
    --namespace kube-system \
    --cluster demo-cluster-three-tier-1 \
    --role-name AmazonEKS_EBS_CSI_DriverRole \
    --role-only \
    --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
    --approve
```
- Run the following command. Replace with the name of your cluster, with your account ID.
```
eksctl create addon --name aws-ebs-csi-driver --cluster demo-cluster-three-tier-1 --service-account-role-arn arn:aws:iam::<AWS-ACCOUNT-ID>:role/AmazonEKS_EBS_CSI_DriverRole --force
```

### Deploy your app with helm
- Create a new namespace
```
kubectl create ns robot-shop
```
- Install the service robot-shop
```
cd three-tier-architecture-demo/EKS/helm
helm install robot-shop --namespace robot-shop .
```
- Verify the services and pods
```
kubectl get pods -n robot-shop
kubectl get svc -n robot-shop
```
- Install the ingress 
```
kubectl apply -f ingress.yaml
```
- Check the Ingress IP address (web interface)
```
kubectl get ingress -n robot-shop
```

> Congratuations! :tada:

- Delete cluster and all components
```
eksctl delete cluster --name demo-cluster-three-tier-1 --region eu-north-1
```
=======
# 3-tier-app
>>>>>>> 7d8507fea48d612958d3fe1e2c842ceb579e80d9
