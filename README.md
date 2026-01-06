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

AmazonSSMManagedInstanceCore

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

## STEP3: configure aws cli on local laptop with accesskey and secretkey
## STPE4: clone github repo
## STPE5: switch to cluster folder, change ec2 key pair name under variables.tf file, create elk cluster using terraform
```bash
terraform apply --auto-approve
```
## STPE6:  update kubeconfig
```bash
aws eks update-kubeconfig --region ap-south-1 --name devopsshack-cluster
```
## Create namespace webapps
```bash
kubectl create namespace webapps
```
## Create Service Account, Role & Assign that role, And create a secret for Service Account and geenrate a Token
### Creating Service Account
### Bellow yml files available under cluster folder
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
## STEP7: launch 2 servers of t2.medium, one for setting up jenkins and another for nexus
### On Jenkins-server: 
- Install docker, jenkins, trivy, kubectl
- create jenkins user
- install suggested plugins
- Install some of the plugins mannually such as
    - Config File Provider
    - Maven Integration
    - Pipeline Maven Integration
    - Docker Pipeline
    - kubernetes
    - kubernetes CLI
    - Pipeline Stage View
## STEP8: Setup tools
- maven --> maven3
## STEP9: Add credentials
- github-cred
- docker-cred
- k8s-token
## STEP10: Install docker on nexus server and run nexus as container
```bash
docker run -d \
  --name nexus \
  -p 8081:8081 \
  sonatype/nexus3
```
## STEP11: Login to nexus contaner and grab the initial password
## STEP12: configure nexus settings under manage jenkins --> managed files --> add a new config --> select Global Maven settings.xml
```xml
<server>
<id>maven-releases</id>
<username>admin</username>
<password>Admin@123</password>
</server>
<server>
<id>maven-snapshots</id>
<username>admin</username>
<password>Admin@123</password>
</server>
```
### STEP13: edit pom.xml
```xml
<distributionManagement>
        <repository>
            <id>maven-releases</id>
            <url>http://13.203.229.39:8081/repository/maven-releases/</url>
        </repository>
        <snapshotRepository>
            <id>maven-snapshots</id>
            <url>http://13.203.229.39:8081/repository/maven-snapshots/</url>
        </snapshotRepository>
    </distributionManagement>
```
### STEP14: Create a jenkins job with some name (blue-green) copy pipeline script from "jenkinsfile_success" paste into pipline section
### STEP15: Verify all env's in pipleine are correct
### STEP16: run the pipeline and cancel it to load parameters
### STEP17: Now choose blue and blue and click on build
### STEP18: Now choose Green and Green and click on build
### STEP19: Now choose Green, Green, switch_traffic and click on build
### STEP20: Verify
```bash
k get all -n webapps
```
## Result: ACCESS THE APPLICATION WITH LOAD BALANCER URL
