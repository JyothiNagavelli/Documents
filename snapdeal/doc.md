
Create eks cluster by using eksctl:

  a. Install eksctl using the following commands
  
  $ curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
  $ sudo mv /tmp/eksctl /usr/local/bin
  $ eksctl version
  
  b. using the following command create eks cluster with forgated nodes
  
  $ eksctl create cluster --name my-cluster --region region-code --fargate
  $ kubectl get nodes -o wide
  $ kubectl get pods -A -o wide
  
  c. for deleting an eks cluster 
 
  $ eksctl delete cluster --name my-cluster --region region-code
 
Add efs to a pod on eks cluster:

 --> for this we need kubectl version >= 1.14
 Pre-requisites:
 
  
  1. Install the AWS CLI.
  2. Set AWS Identity and Access Management (IAM) permissions for creating and attaching a policy to the Amazon EKS worker node role CSI    Driver Role.
  3. Create your Amazon EKS cluster and join your worker nodes to the cluster.
  
  

Deploy and test the Amazon EFS CSI driver


 a. Download the IAM policy document from GitHub:
 
  $ curl -o iam-policy-example.json https://raw.githubusercontent.com/kubernetes-sigs/aws-efs-csi-driver/v1.2.0/docs/iam-policy-example.json
  
  
  b. Create an IAM policy:
  
  
  $ aws iam create-policy \
    --policy-name AmazonEKS_EFS_CSI_Driver_Policy \
    --policy-document file://iam-policy-example.json
  
  
  c.Annotate the Kubernetes service account with the IAM role ARN and the IAM role with the Kubernetes service account name. For example:
 
   $ aws eks describe-cluster --name your_cluster_name --query "cluster.identity.oidc.issuer" --output text
 
  d. Create the following IAM trust policy, and then grant the AssumeRoleWithWebIdentity action to your Kubernetes service account.
  
  
~cat <<EOF > trust-policy.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::YOUR_AWS_ACCOUNT_ID:oidc-provider/oidc.eks.YOUR_AWS_REGION.amazonaws.com/id/<XXXXXXXXXX45D83924220DC4815XXXXX>"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "oidc.eks.YOUR_AWS_REGION.amazonaws.com/id/<XXXXXXXXXX45D83924220DC4815XXXXX>:sub": "system:serviceaccount:kube-system:efs-csi-controller-sa"
        }
      }
    }
  ]
}
EOF


e.Create an IAM role:


~ aws iam create-role \
  --role-name AmazonEKS_EFS_CSI_DriverRole \
  --assume-role-policy-document file://"trust-policy.json"
  
  
f. Attach your new IAM policy to the role:


~ aws iam attach-role-policy \
  --policy-arn      arn:aws:iam::<AWS_ACCOUNT_ID>:policy/AmazonEKS_EFS_CSI_Driver_Policy \
  --role-name AmazonEKS_EFS_CSI_DriverRole



g. Create a Kubernetes service account that's annotated with the ARN of the IAM role that you created. For example:


~ 
  cat << EOF > efs-service-account.yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: efs-csi-controller-sa
  namespace: kube-system
  labels:
    app.kubernetes.io/name: aws-efs-csi-driver
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::YOUR_AWS_ACCOUNT_ID:role/AmazonEKS_EFS_CSI_DriverRole
EOF


h. Apply the manifest:

  ~ kubectl apply -f efs-service-account.yaml


Deploy the Amazon EFS CSI driver:

The Amazon EFS CSI driver allows multiple pods to write to a volume at the same time with the ReadWriteMany mode.

 
i.To deploy the Amazon EFS CSI driver, run one of the following commands based on your Region or cluster type: 
  
All Regions other than China Regions:
 
~ kubectl apply -k "github.com/kubernetes-sigs/aws-efs-csi-driver/deploy/kubernetes/overlays/stable/ecr/?ref=release-1.1"

Beijing and Ningxia China Regions:


~ kubectl apply -k "github.com/kubernetes-sigs/aws-efs-csi-driver/deploy/kubernetes/overlays/stable/?ref=release-1.1"

If your cluster contains only AWS Fargate pods (no nodes), then deploy the driver with the following command (all Regions):


~ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-efs-csi-driver/master/deploy/kubernetes/base/csidriver.yaml


ii. Get the VPC ID for your Amazon EKS cluster:

~ aws eks describe-cluster --name your_cluster_name --query "cluster.resourcesVpcConfig.vpcId" --output text



iii. Get the CIDR range for your VPC cluster:


~ aws ec2 describe-vpcs --vpc-ids YOUR_VPC_ID --query "Vpcs[].CidrBlock" --output text

iv. Create a security group that allows inbound network file system (NFS) traffic for your Amazon EFS mount points:


~ aws ec2 create-security-group --description efs-test-sg --group-name efs-sg --vpc-id YOUR_VPC_ID

v.  Add an NFS inbound rule so that resources in your VPC can communicate with your Amazon EFS file system:

~ aws ec2 authorize-security-group-ingress --group-id sg-xxx --protocol tcp --port 2049 --cidr YOUR_VPC_CIDR


vi.  Create an Amazon EFS file system for your Amazon EKS cluster:


~ aws efs create-file-system --creation-token eks-efs


vii. To create a mount target for Amazon EFS, run the following command:


~ aws efs create-mount-target --file-system-id FileSystemId --subnet-id SubnetID --security-group sg-xxx


Test the Amazon EFS CSI driver:


Reference: 
   

    ~ git clone https://github.com/kubernetes-sigs/aws-efs-csi-driver.git

  ~ cd aws-efs-csi-driver/examples/kubernetes/multiple_pods/

Retrieve your Amazon EFS file system ID that was created earlier:

 
~ aws efs describe-file-systems --query "FileSystems[*].FileSystemId" --output text


In the specs/pv.yaml file, replace the spec.csi.volumeHandle value with your Amazon EFS FileSystemId from previous steps.

 Create the Kubernetes resources required for testing:

~  kubectl apply -f specs/

~ kubectl get pv -w

~ kubectl describe pv efs-pv

PROMETHEUS:

Install prometheus on a cluster using helm using the following command:

~   helm install prometheus stable/prometheus-operator



https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack
            
                                                ( or )
For getting alerts we can deploy alertmanager with different values.yaml and rules.yaml. For that we can refer the following link.

https://medium.com/techno101/how-to-send-a-mail-using-prometheus-alertmanager-7e880a3676db

CREATE AN EKS CLUSTER USING TERRAFORM:


Install terraform on ubuntu using the folowing commands:

Commands:

~ curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
~ sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
~ sudo apt-get update && sudo apt-get install terraform


Initiate:
~ terraform init

Formate:
~ terraform fmt

Plan:
~ terraform plan

Validate:
~ terraform validate

Apply:
~ terraform apply –auto-approve

Delete:
~ terraform destroy –auto-approve
To create eks cluster using terraform please refer the following link:

 ~ https://github.com/hashicorp/learn-terraform-provision-eks-cluster

Deploy loki on cluster and test :

https://www.youtube.com/watch?v=UM8NiQLZ4K0&t=1s


Alerts on loki:

https://opsmatters.com/videos/alert-your-loki-logs-grafana

Installation:

we can install loki by using helm:
  

~ helm repo add grafana https://grafana.github.io/helm-charts
~ helm repo update
~ helm search repo grafana

Find the grafana/loki-stack this is latest version and we need this for loki to get logs from cluster.
We bneed to know what is configured in the values.yaml file. From that we can configure our actual required chart by change the values in values.yaml. Finally install the helm from our customised values.yaml.

~ helm show values grafana/loki-stack > loki-stack.yaml

>> values.yaml
~ loki: 
      enabled: true
      persistence:
          enabled: true
          size: 1Gi
   promtail:
        enabled: true
   grafana:
       enabled: true
       sidecar:
            datasources:
                 enabled: true
            image:
               tag: 7.5.0
~ helm install loki-stack grafana/loki-stack -n loki –create-namespace –values values.yaml
Finally we installed loki in our cluster in loki namespace. Now check all the components in loki using the following commands.

~ kubectl get all -n loki

For getting the loki UI port-forward the svc by using the following command

 ~ kubectl port-forward svc/loki-stack-grafana -n loki 3000:80

Using the address obtained by above command , now we are able to login to the grafana UI.
By default grafana user name is admin and by run the following command we will get the password.

~ kubectl get secret --namespace <YOUR-NAMESPACE> loki-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo


Now we got the password for grafana, login to the grafana and check our logs and all what ever we want. We can create alerts directly in UI.

WwwemfmkfjkrejreklgjtkergmjfjklgsdrjigpFFFFTTGQFGFVQAGZSFVAG
Build And Deploy OurApplication On Gitlab

Scenario:
   GitLab includes utilities that allows us to create pipelines that can be used for CI and CD. We will be creating a pipeline for a .net application. The pipeline will do 2 functions.

    1. Build
    2. Deploy

Prerequisites:
    1. GitLab
    2. Docker Hub and GCR
    3. Cluster

The only requirements to implement GitLab CI/CD with a project are:
    1. A project hosted in a GitLab repository to set up your CI/CD in GitLab.
    2. A .gitlab-ci.yml file at the root of your project directory. .gitlab-ci.yml contains the project-specific build and deployment scripts that you need in your pipeline. An individual script used in a .gitlab-ci.yml is called a job. There can be multiple jobs in a CI/CD pipeline.
       3. A runner configured in GitLab. A GitLab runner is runs the jobs in .gitlab-ci.yml.


Runner:
 We need to create runner to run the job in the .gitlab-ci.yml. For that we need repository registration token. You will find this token in settings > CI/CD > runners.
Install Runner binary for system:
```
$ sudo curl -L --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64
$ sudo chmod +x /usr/local/bin/gitlab-runner
$ sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash
$ sudo gitlab-runner install --user=gitlab-runner –working-directory=/home/gitlab-runner
$ sudo gitlab-runner start
 
```
Create runner with this command
```
$ sudo gitlab-runner register \
--non-interactive \
--url "https://gitlab.com/" \
--registration-token "<registration token>" \
--executor "docker" \
--docker-image alpine:latest \
--description "docker-runner" \
--maintenance-note "Free-form maintainer notes about this runner" \
--tag-list "docker,aws" \
--run-untagged="true" \
--locked="false" \
--access-level="not_protected"
```
After completion of creation check it on GitLab in runners section and make sure you disable shared runner option. then only GitLab use our own runners.

.gitlab-ci.yml:
Here we are using Kaniko for building docker image and pushed to Docker Hub. So, For that we need to specify Docker Hub credentials as a Variables. These variables we are passing inside the .gitlab-ci.yml. GitLab have facility to save variable here settings > CI/CD > Variables.
```
CI_REGISTRY : https://index.docker.io/v1/
CI_REGISTRY_USER : <username>
CI_REGISTRY_PASSWORD : <password>
CI_REGISTRY_IMAGE : <username>/<imagename>

```
Now create .gitlab-ci.yml file with Kaniko-executer. For more information about Kaniko check here.
https://docs.gitlab.com/ee/ci/docker/using_kaniko.html#building-a-docker-image-with-kaniko

 tIn .gitlab-ci.yml we have stages. In stages we are mentioning our jobs list. kaniko uses the Dockerfile under the root directory of the project, builds the Docker image and pushes it to the Docker Hub. make sure you have Dockerfile under the root directory. My Dockerfile looks like this
 ```
FROM nginx:alpine
LABEL author="jyothi"
COPY index.html /usr/share/nginx/html/index.html
```
Build job:
  For build and push purpose we are using kaniko. So we need kaniko job template. like this
```
stages:
 - build
build:
stage: build
image:
name: gcr.io/kaniko-project/executor:debug
entrypoint: [""]
script:
- mkdir -p /kaniko/.docker
- echo "{\"auths\":{\"${CI_REGISTRY}\":{\"auth\":\"$(printf "%s:%s" "${CI_REGISTRY_USER}" "${CI_REGISTRY_PASSWORD}" | base64 | tr -d '\n')\"}}}" > /kaniko/.docker/config.json
- cat /kaniko/.docker/config.json
- >-
/kaniko/executor
--context "${CI_PROJECT_DIR}"
--dockerfile "Dockerfile"
--destination "${CI_REGISTRY_IMAGE}:${CI_IMAGE_TAG}"
```
So, This job build and push our image in to Docker Hub with use of Credentials as variables which we created. pipeline uses those credentials.
                                                  (or)
You can use docker-dind service instead of kaniko it will work. Here we will specify GitLab access token as a variable like CI_GITLAB_TOKEN. we will use it instead of the password
```
build:
  image: docker:latest
  stage: build
  services:
    - docker:dind
  before_script:
    - echo "$CI_GITLAB_TOKEN" | docker login -u $CI_REGISTRY_USER --password-stdin $CI_REGISTRY
  script:
    - docker info
    - docker build --pull -t abhi1-rithu2/first . 
- docker push registry.gitlab.com/abhi1-rithu2/first
Now lets use kaniko for our project:

.gitlab-ci.yml file
```
stages:
   - build
build:               # job name
  stage: build
  image:
    name: gcr.io/kaniko-project/executor:debug
    entrypoint: [""]
  script:
    - mkdir -p /kaniko/.docker
    - echo "{\"auths\":{\"${CI_REGISTRY}\":{\"auth\":\"$(printf "%s:%s" "${CI_REGISTRY_USER}" "${CI_REGISTRY_PASSWORD}" | base64 | tr -d '\n')\"}}}" > /kaniko/.docker/config.json
    - cat /kaniko/.docker/config.json
    - >-
      /kaniko/executor
      --context "${CI_PROJECT_DIR}"
      --dockerfile "Dockerfile"
    --destination  "${CI_REGISTRY_IMAGE}:tag"
After commit the changes pipeline will starts. We can find pipelines inside the CI/CD section. There we have few options like pipelines, jobs etc.
For GitLab container registry:
Do same process for GitLab container registry. In GitLab every project have own container registry to save docker images instead of other registries. You can find this here Package & Registries > container registry.

Deploy job:
So, After building and pushing docker images into registries we need to do deployment for our application by using docker images. we will deploy our application on Kubernetes by using ArgoCD and K3s cluster.
steps:-
- Install K3s cluster on your machine here.
- Install ArgoCD and ArgoCD CLI on cluster here.
- Set cluster config file as a variable in GitLab.
- Add deploy job in .gitlab-ci.yaml file.


Cat cluster config file:
~ cat /etc/rancher/k3s/k3s.yaml
K3s cluster works with <localhost:6443>. Localhost did not work at GitLab. for that we will do ngrok proxy with 6443 port. Its give access to use cluster config in GitLab. Install ngrok here.
   https://ngrok.com/download
After installation do like this:
 
~ ngrok tcp 6443

We get a ngrok url. this URL we will use instead of localhost<127.0.0.1:6443> inside the config file. we need to do another thing, encode the config file and set as a variable in GitLab.

Encode the config file with this command.

~ echo $(cat <config file path>| base64) | tr -d " "

Copy the content and create a variable in GitLab name as KUBE_CONFIG. Before adding the deploy stage into .gitlab-ci.yaml, Setup ArgoCD UI.

Now, add deploy job in .gitla-ci.yaml
````
stages:
   - build
   - deploy
build:               # job name
  stage: build
  image:
    name: gcr.io/kaniko-project/executor:debug
    entrypoint: [""]
  script:
    - mkdir -p /kaniko/.docker
    - echo "{\"auths\":{\"${CI_REGISTRY}\":{\"auth\":\"$(printf "%s:%s" "${CI_REGISTRY_USER}" "${CI_REGISTRY_PASSWORD}" | base64 | tr -d '\n')\"}}}" > /kaniko/.docker/config.json
    - cat /kaniko/.docker/config.json
    - >-
      /kaniko/executor
      --context "${CI_PROJECT_DIR}"
      --dockerfile "Dockerfile"
      --destination  "${CI_REGISTRY_IMAGE}:tag"
deploy:
  stage: deploy
  image: rithu1320/first-gitlab
  script:
     - echo $KUBE_CONFIG | base64 -d  > /tmp/kube_config.yaml 
     - export KUBECONFIG=/tmp/kube_config.yaml 
     - kubectl apply -f appset.yaml -n argocd   
  dependencies:
     -  build
  when: manual
```
Summary:
Thus, we can create GitLab CI/CD pipeline for create docker images and pushed into Docker Hub and GitLab container registry, and by using those images deploy our application on kubernetes cluster through ArgoCD.
