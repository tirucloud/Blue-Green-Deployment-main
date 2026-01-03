## Blue Green Deployment

## STEP1: Install awscli, kubectl on laptop
# AWSCLI

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
aws configure
```

## KUBECTL

```bash
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```
## STPE2: create a IAM user with all required permissions
## Attach Policies to the newly created user
## below policies
AmazonEC2FullAccess

AmazonEKS_CNI_Policy

AmazonEKSClusterPolicy	

AmazonEKSWorkerNodePolicy

AWSCloudFormationFullAccess

IAMFullAccess

#### One more policy we need to create with content as below
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "eks:*",
            "Resource": "*"
        }
    ]
}
```
Attach this policy to your user as well

![Policies To Attach](https://github.com/jaiswaladi246/EKS-Complete/blob/main/Policies.png)

## STEP3: confugre aws cli on local laptop with acceskye and secretekey
## STPE4: clone github repo
## STPE5: switch to cluster folder, change ec2 key pair name under main.tf file, create elk cluster using terraform
```bash
terraform apply --auto-approve
```
## STPE6:  update kubeconfig
## 
```bash
aws eks update-kubeconfig --region ap-south-1 --name my-eks23

```
## Create namespace webapps
```bash
kubectl create namespace webapps
```
## Create Service Account, Role & Assign that role, And create a secret for Service Account and geenrate a Token
### Creating Service Account
```bash
k apply -f sa.yml
```

### Create Role
```bash
k apply -f role.yml
```
### Bind the role to service account
```bash
k apply -f binding.yml
```
### Generate token using service account in the namespace
```bash
k apply -f secret.yml -n webapps
```
[Create Token](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/#:~:text=To%20create%20a%20non%2Dexpiring,with%20that%20generated%20token%20data.)
```bash
k describe secret mysecretname -n webapps
```
