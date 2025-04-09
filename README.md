After creating EKS Cluster

Update eks namespace in k8s_manifests folder frontend, backend and full stack yaml files.

Update ECR images in backend and frontend deployment files.

Update eks namespace in deploy,serviceand secrets yaml files in k8s_manifests/mongo.


Step 1 →Setup aws load balancer ; installation and attachement it to your EKS cluster
1. Below command fetch the iam policy for your ALB

curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json

2. This command create the iam policy in your aws account from iam_policy.json file that is setup in the first command

aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json

3. This command apply the load balancer policy to your eks cluster so that your eks cluster is working with your load balancer according to the policy

eksctl utils associate-iam-oidc-provider --region=us-east-1 --cluster=<cluster-name> --approve

4. This command create and attach an service account to your cluster so that your cluster is allowed to work with load balancer service

please change your aws account no. from the below command otherwise it won’t work

eksctl create iamserviceaccount --cluster=<cluster-name> --namespace=<namespace> --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::<Account No>:policy/AWSLoadBalancerControllerIAMPolicy --approve --region=us-east-1

All the policies are attached let’s deploy the load balancer
5. For this we have to install helm→Helm is a special tool that helps you easily carry and manage your software when you’re using Kubernetes, which is like a big playground for running applications.

sudo snap install helm --classic
6. After this we have to add a particular manifest for load balancer that is pre written by someone on eks repo by using helm

helm repo add eks https://aws.github.io/eks-charts
7. update the eks repo using helm

helm repo update eks
8. Install the load balancer controller on your eks cluster

helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n <namespace> --set clusterName=<cluster-name> --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
kubectl get deployment -n <namespace> aws-load-balancer-controller

