                                                             ArgoCD
Introduction:
      
    ArgoCD is declarative, GitOps is a continuous delivery tool for Kubernetes.
ArgoCD follows the GitOps pattern of using Git repositories as the source of truth for defining the desired application state. Kubernetes manifests can be specified in several ways:
    • Kustomize applications
    • Helm charts
    • Plain directory of YAML/JSON manifests
    • Any custom config management tool configured as a config management plugin



    • Pre-requisites:

      . Clusters
      . Gitlab
      . Helm
   
Initialising the Cluster:

For this project, the developer uses the k3s clusters. To initialise the cluster use the following commands.
                curl -sfL https://get.k3s.io | sh -
                export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
                sudo chmod 644 $KUBECONFIG
Gitlab:

 Developer need to be keep the source code and k8s manifests in the gitlab repository.

ArgoCD Installation:
    
For installing argocd in cluster use the following commands:
kubectl create namespace argocd
kubectl apply -n argocd -f   https://raw.githubusercontent.com/argoproj/argo-cd/v2.2.3/manifests/install.yaml

    • The developer needs to edit the service for accessing the argocd UI ,because by     default argocd-server is of service type ClusterIP ,so its not accessible for outside.
          Using the following command to see all the components in argocd 
      
           $ kubectl get all -n argocd

. Using the following command edit the argocd-server service.The service type will be either NodePort or LoadBalancer:
             
$ kubectl get svc -n argocd
$ kubectl edit service/argocd-server -n argocd
 
. Now developer able to access the argocd UI via web using:

            ex: https://192.168.1.1:Nodeport(30001)
 
. Now login to the Argocd UI

            username: admin
            password: ******

 . Get the password by using the following commands:

        $ kubectl get secrets -o yaml

 . From the above command we get the password and lets decode it by using the     following command:

         $ echo ‘password’ | base64 –decode

. And the username is admin by default.
. Finally able to login argocd UI successfully
. Its ready to deploy the application. Applications can be deployed using application set,CRD and from UI also.

ApplicationSet:                                                                                                                                                                                                                                                                                                                                                                               
      
Through applicationset file ,the developer can deploy single application on multiple clusters and multiple applications on single/multiple clusters.
So that, the developer needs to install applicationset to apply applicationset file on clusters. Using the command:

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj-labs/applicationset/v0.3.0/manifests/install.yaml

for ex: To deploy sample application

apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: guestbook
spec:
  generators:
  - list:
      elements:
      - cluster: finance-preprod
        url: https://9.8.7.6
  template:
    metadata:
      name: '{{cluster}}-guestbook'
    spec:
      project: default
      source:
        repoURL: https://github.com/argoproj/applicationset.git
        targetRevision: HEAD
        path: examples/list-generator/guestbook/{{cluster}}
      destination:
        server: '{{url}}'
        namespace: guestbook
Now apply the applicationset file using the following command:

   $kubectl apply -f <fileneme> -n argocd

. The developer needs to deploy application on multiple clusters ,then needs to add multiple clusters to argocd. From UI there is no options for add clusters to it.
So  the developer should install argocd CLI for getting more options in argocd ,login to argocd from CLI and deploy apps and add clusters and more options here.
. By using the following commands:

$ wget https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
$ sudo mv argocd-linux-amd64 /usr/local/bin/argocd
$ chmod +x /usr/local/bin/argocd
$ argocd –help

Login to argocd via CLI:
   
. By following commands login to argocd from CLI:

      $ argocd login <node ip>:<NodePort>

. Update password using following commands:

     $ argocd account update-password 

. Create application and more options in that :

    $ argocd create app –help
    $ argocd app list
    $ argocd app delete <name>

. Add clusters to argocd using following commands:
    $ argocd cluster add Default --name=<name> --kubeconfig <pwd/kluster.yaml>
     $ argocd cluster list
     $ argocd cluster rm <name or url>

 For ex: deploying application on multiple clusters
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: guestbook
spec:
  generators:
  - list:
      elements:
      - cluster: engineering-dev
        url: https://1.2.3.4
      - cluster: engineering-prod
        url: https://2.4.6.8
      - cluster: finance-preprod
        url: https://9.8.7.6
  template:
    metadata:
      name: '{{cluster}}-guestbook'
    spec:
      project: default
      source:
        repoURL: https://github.com/argoproj/applicationset.git
        targetRevision: HEAD
        path: examples/list-generator/guestbook/{{cluster}}
      destination:
        server: '{{url}}'
        namespace: guestbook

Ex2: Deploying application with different parameters on multiple clusters.
      

apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: test
spec:
  generators:
  - list:
      elements:
      - cluster: in-cluster
        url: https://kubernetes.default.svc
        values:
          replicaCount: '2'
      - cluster: argo
        url: https://vc-t8lnrrytj8fwb63clf49.zlr-dc1.apps.dev.klusternetes.com
        values:
          replicaCount: '3'
      - cluster: argo2
        url: https://vc-7keockut9teuzqzp9cyx.zlr-dc1.apps.dev.klusternetes.com
        values:
          replicaCount: '4'
  template:
    metadata:
      name: '{{cluster}}-test'
    spec:
      project: default
      source:
        repoURL: https://github.com/argoproj/project.git
        targetRevision: HEAD
        path: Ezy/project-test
        helm:
          parameters:
          - name: replicaCount
            value: '{{values.replicaCount}}'

      destination:
        server: '{{url}}'
        namespace: default

Ex3: To sync automatically
     



Summary:
  Thus , the developer can able to deploy applications on multiple clusters using gitops tool argocd.Argocd is easy to adapt for deployments in k8s. It won’t allow manual changes, the only trust is git repository.In argocd the app deployment is transparent.

Add multiple clusters from multiple systems(VMs):

. By using ngrok or ingress connect multiple vms clusters.
. Now using ngrok connect multiple clusters by using following commands:
  
$ ngrok tcp 6443(cluster port)

. Now we copy the url after run above command and paste it in kluster.yaml at server:<url>.
. Then add cluster to argocd by using above commands.
