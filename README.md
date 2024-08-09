# Three-Tier Application Deployment
In this application we have a Three-tier Web Application created using React JS, Node JS, MongoDB. And we are deploying this application using AWS EKS. 

Technologies used:
1. Application: React JS, Node JS, MongoDB
2. AWS: EC2, IAM, ECR, EKS
3. Containerzation: Docker, EKS
4. Registry: ECS  
5. Version control: Git
6. Kubernetes manifests files using YAML

STEPS:  
1. Create an IAM User:   
   * Create an IAM User with AdministratorAccess and generate Access Key and Secrete Access Key.

2. Create EC2:   
   * Create an EC2 machine and attach required security groups.  
   * SSH into the machine

3. Install AWS CLI:   
   ```
   curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"  
   sudo apt install unzip  
   unzip awscliv2.zip   
   sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update  
   aws configure
4. Install docker:   
   ```
   sudo apt-get update  
   sudo apt install docker.io  
   docker ps  
   sudo chown $USER /var/run/docker.sock  
5. Install kubectl:   
   ```
   curl -o kubectl https://amazon-eks.s3.us-east-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl  
   chmod +x ./kubectl  
   sudo mv ./kubectl /usr/local/bin  
   kubectl version --short --client  

6. Install eksctl:   
   ```
   curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp  
   sudo mv /tmp/eksctl /usr/local/bin  
   eksctl version  

7. Setup EKS cluster:   
   ```
   eksctl create cluster --name three-tier-cluster --region us-east-2 --node-type t2.medium --nodes-min 2 --nodes-max 2  
   aws eks update-kubeconfig --region us-east-2 --name three-tier-cluster  
   kubectl get nodes  

8. Run kubernet manifests:   
   ```
   kubectl create namespace workshop  
   kubectl apply -f .   

9. Install AWS Load Balancer:   
   ```
   #Download IAM policy json file for AWS Load Balancer Controller to manage load balancers:   
     curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json  
   #Creating a policy that grants the necessary permissions for the Load Balancer Controller to operate:  
     aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json  
   #OIDC provider allows Kubernetes service accounts to assume IAM roles:  
     eksctl utils associate-iam-oidc-provider --region=us-east-2 --cluster=three-tier-cluster --approve  
   #Create an IAM service account in the kube-system namespace called aws-load-balancer-controller. The service account is linked to an IAM role (AmazonEKSLoadBalancerControllerRole) that have the AWSLoadBalancerControllerIAMPolicy attached to grant the necessary permissions to manage ELBs:  
     eksctl create iamserviceaccount --cluster=three-tier-cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::YOUR-AWS-ACCOUNT-ID:policy/AWSLoadBalancerControllerIAMPolicy --approve --region=us-east-2
     
10. Deploy AWS Load Balancer Controller: 
    ```
    #Install helm which work as package manager for Kubernetes:  
     sudo snap install helm --classic   
     helm repo add eks https://aws.github.io/eks-charts  
     helm repo update eks  
    #This command deploys the AWS Load Balancer Controller into the kube-system namespace of your Kubernetes cluster:
     helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=my-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller  
     kubectl get deployment -n kube-system aws-load-balancer-controller  
     kubectl apply -f ingress.yaml  
