## Build and Deploy a tomcat based Application using Maven and ArgoCD.

 ![Automated CI-CD](https://github.com/Rojha-git/tomcat-app/blob/main/images/dev1finle.png)


## Table of Contents
- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Step-by-Step Guide](#step-by-step-guide)
  - [1. Create an EC2 Instance on AWS](#1-create-an-ec2-instance-on-aws)
  - [2. Install Terraform](#2-install-terraform)
  - [3. Clone the Terraform Scripts](#3-clone-the-terraform-scripts)
  - [4. Check the Status of Installed Services](#4-check-the-status-of-installed-services)
  - [5. Login to Jenkins Console](#5-login-to-jenkins-console)
  - [6. Configure Jenkins and Install Plugins](#6-configure-jenkins-and-install-plugins)
  - [7. Generate Access Tokens](#7-generate-access-tokens)
  - [8. Create Jenkins Pipeline](#8-create-jenkins-pipeline)
  - [9. SonarQube Configuration](#9-sonarqube-configuration)
  - [10. Setup Monitoring with Prometheus and Grafana](#10-setup-monitoring-with-prometheus-and-grafana)
  - [11. Integrate Gmail with Jenkins](#12-Integrate-Gmail-with-Jenkins)
  - [12. Jenkins Pipeline Execution](#11-jenkins-pipeline-execution)
  - [13. Create AWS EKS Cluster](#12-create-aws-eks-cluster)
  - [14. Integrate Prometheus with EKS and Import Grafana Dashboard](#13-integrate-prometheus-with-eks-and-import-grafana-dashboard)
  - [15. Automated Deployment with ArgoCD on Kubernetes](#14-automated-deployment-with-ArgoCD-on-kubernetes)
  - [16. Webhook Configuration](#15-webhook-configuration)
- [Conclusion](#conclusion)
- [Acknowledgements](#acknowledgements)
- [License](#license)

## Introduction
This repository provides a step-by-step guide for building and deploying a modern YouTube clone application using React JS with Material UI 5. The guide covers setting up infrastructure on AWS, configuring Jenkins for CI/CD, integrating SonarQube for code analysis, and implementing monitoring with Prometheus and Grafana.

## Prerequisites
Before you begin, make sure you have the following prerequisites:
- AWS account
- GitHub account
- DockerHub account
- SonarQube account
- Gmail account for email notifications

## Step-by-Step Guide

1.create an ec2(t2.micro) instance on AWS and after ssh to this instance install terraform on that using below commands:
    #Confirm the latest version number on the terraform website:
    https://www.terraform.io/downloads.html
    
      wget https://releases.hashicorp.com/terraform/1.0.7/terraform_1.0.7_linux_amd64.zip

      apt install unzip
    
      unzip terraform_1.0.7_linux_amd64.zip
    
      sudo mv terraform /usr/local/bin/
    
      terraform --version


2.clone the git repo for terraform scripts-


      git clone "https://github.com/Rojha-git/Terraform_YT_cloned_app.git"
   
      cd Terraform_YT_cloned_app        #under this we have two repo so choose one by one and run below command.
   
      terraform init                    # aws should be configure with your instances
   
      terraform plan
   
      terraform apply
   
    
3.check the status of the installed services:


   check jenkins [http://<ip_addr_jenkins_server>:8080](http://<ip_addr_jenkins_server>:8080)

   check sonar: http://<ip_addr_jenkins_server>:9000
   
   check prometheus: http://<ip_addr_monitoring_server>:9090  #if url is not came up check status of service if not running start it using

   ```bash

     sudo systemctl start prometheus

   ```
   check grafana []http://<ip_addr_monitoring_server>:3000

   --Commands to check services

   ```bash
   
    sudo systemctl status prometheus
   
    sudo systemctl status node_exporter
   
    sudo systemctl status grafana-server

   ```

4.login to jenkins console:
   
   using http://<ip_addr_jenkins_server>:8080
   
   primary password you can copy from the server using below commad :
   ```bash

   sudo cat /var/lib/jenkins/secrets/initialAdminPassword

   ``` 
   - Enter the Administrator password
   
   after that you can create your userid and password.

   ![jenkins-console](https://user-images.githubusercontent.com/43399466/215959008-3ebca431-1f14-4d81-9f12-6bb232bfbee3.png)

5. after login to jenkins first of all click on "manage plugins" and install required plugins for **"Docker, Sonar, maven , eclipse, quality gates , prometheus"** and restart jenkins.
   also under the "tools" section configure jdk , maven , sonar scanner and
   under "system>>sonarqube-installation>> "configure sonar server using http://<privateip_sonar_server>:9000 console urls.
   
   under "manage jenkins>>tools>>sonarqube installation>>" configure sonar tool with latest version .

   under "manage jenkins>>tools>>" Configure Maven and jdk as tools.

7. Generate the access token from sonarqube(secret_text) , github , dockerhub and configure all this under the jenkins "credentials" so that jenkins can access all these.

8. Under jenkins create an pipeline with "tomcat-app-ci-cd" name using below steps:

     --- select SCM option and provide repo [https://github.com/Rojha-git/tomcat-app.git](https://github.com/Rojha-git/tomcat-app.git) to access jenkins file for ci-cd.

9. login to sonarqube console and configure quality gate , webhook , once all these completed then after the jenkins pipeline 
    completion you will be able to see all the flow and unit tests.

   ![sonar-project-overview](https://github.com/Rojha-git/tomcat-app/blob/main/images/sonar1.png)
   ![sonar-to-check-the-code-issue](https://github.com/Rojha-git/tomcat-app/blob/main/images/sonar2.png)
   ![sonar-overview](https://github.com/Rojha-git/tomcat-app/blob/main/images/sonar3.png)

11. I--Login to prometheus using http://<ip_addr_monitoring_server>:9090   
    after login :
    
    --Add job for node exporter in prometheus
   
    ```bash
    
     cd /etc/prometheus/
   
     sudo nano prometheus.yml

    ```
    #add job for node exporter abd jenkins
         
    
         - job_name: 'node_exporter'
           static_configs:
             - targets: ['IP-Address-monitoring:9100']

    
         - job_name: 'jenkins'
           metrics_path: '/prometheus'
           static_configs:
             - targets: ['IP-Address-jenkins:8080']
    
         
   
    ```bash
    
    #Check the indentatio of the prometheus config file with below command
    
      promtool check config /etc/prometheus/prometheus.yml

    #Reload the Prometheus configuration
    
      curl -X POST http://localhost:9090/-/reload

    ```
    **After performing the #10 point you will be able to see the targets for matrices under the traget option in prometheus console.**

     II.--login to grafana using http://<ip_addr_monitoring_server>:3000 --> #username and password will be "admin"

    --configure prometheus as the data source under the grafana using the prometheus url:

    --Import or add the dashboard for prometheus and jenkins by dashboard id of grafana **(1860 -- node_exporter , 9964 --jenkins)** # you can try by google as well for dashboard id.


    **** Now you are able to access the monitoring console on grafana for both jenkins job and node_exporter server  ****
    
12. configuration steps for generating gmail report --->>

    --login to your gmail account and search for app password under : " Account >> security " and genaerte the token

    --Go to jenkins console and configure this in jenkins credential using username and password(token).

    --under "Dashboard > Manage jenkins > system" configure email notification using smtp server "smtp.gmail.com" , port "465" and your gmail.

    --similliarly configure extended email notification using global credential that we have configured in jenkins using the same port and server as above.

      **Once you did this above configuration #13 try to test your pipeline**

13. try to test your jenkins pipeline,after successfully completion it will update the tags under(https://github.com/Rojha-git/tomcat-app-cd.git) in deployment.yaml.

    also it will trigeer an mail to mailid that you will provided.

    ![Jenkins-pipeline-status](https://github.com/Rojha-git/tomcat-app/blob/main/images/jenkins.png)

15. Create AWS EKS Cluster
    
    I.--Install kubectl on Jenkins Server
 
          sudo apt update
    
          sudo apt install curl
    
          curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
 
          sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

          kubectl version --client

    II. --Install AWS Cli
    
          curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

          sudo apt install unzip

          unzip awscliv2.zip

          sudo ./aws/install

          aws --version

    III. --Installing  eksctl

          curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
       
          cd /tmp
       
          sudo mv /tmp/eksctl /bin

          eksctl version

    IV.--Setup Kubernetes using eksctl
    
          eksctl create cluster --name tomcatbox-cluster \
    
          --region ap-south-1 \
    
          --node-type t2.small \
    
          --nodes 3 \

    V.--Verify Cluster with below command

          kubectl get nodes    


14.   Refer---https://argo-cd.readthedocs.io/en/stable/cli_installation/

   .#ArgoCD Installation on Kubernetes Cluster and Add EKS Cluster to ArgoCD
   
   1 ) First, create a namespace

       kubectl create namespace argocd

   2 ) Next, let's apply the yaml configuration files for ArgoCd
   
       kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

   3 ) Now we can view the pods created in the ArgoCD namespace.
   
       kubectl get pods -n argocd

   4 ) To interact with the API Server we need to deploy the CLI:
   
       sudo curl --silent --location -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/v2.4.7/argocd-linux-amd64
       
       sudo chmod +x /usr/local/bin/argocd
      
   5 ) Expose argocd-server
   
       kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

   6 ) Wait about 2 minutes for the LoadBalancer creation
    
       kubectl get svc -n argocd

   7 ) Get pasword and decode it and login to ArgoCD "using LB url by above command" on Browser. Go to user info and change the password
     
       kubectl get secret argocd-initial-admin-secret -n argocd -o yaml     #Copy password from here
    
       echo WXVpLUg2LWxoWjRkSHFmSA== | base64 --decode        #decode the password and login argocd username will be admin.

   8 ) login to ArgoCD from CLI
   
       argocd login a2255bb2bb33f438d9addf8840d294c5-785887595.ap-south-1.elb.amazonaws.com --username admin,    provide the password which you set above

   9 ) Check available clusters in ArgoCD
   
       argocd cluster list

   10 ) Below command will show the EKS cluster details
   
        kubectl config get-contexts

   11 ) Add above EKS cluster to ArgoCD with below command
   
        argocd cluster add i-08b9d0ff0409f48e7@tomcatbox-cluster.ap-south-1.eksctl.io --name virtualtechbox-eks-cluster
     
   12 ) Now if you give command "$ argocd cluster list" you will get both the clusters EKS & AgoCD(in-cluster). This can be verified at ArgoCD Dashboard.

   
   13 ) After login to argocd click on create/Add App and provide CD repo url (https://github.com/Rojha-git/tomcat-app-cd.git) ,namespace will be default and slect the cluster by dropdown , path "./" you can provide.

   ![argoCD-App](https://github.com/Rojha-git/tomcat-app/blob/main/images/argo1.png)

   ![argoCD-Overview](https://github.com/Rojha-git/tomcat-app/blob/main/images/argo2.png)


     **once you did all above steps you will be able to see project in argocd with all the stages of deployment**
      
     **run "kubectl get svc" and browse dns followed by port :8080 and by :8080/webapp/ , you will be able to see our finle application that we have deployed**.


15. Integrate Prometheus with EKS and Import Grafana Monitoring Dashboard for Kubernetes

    1--Install Helm
    
        sudo snap install helm --classic
        helm version

    2--Install Prometheus on EKS
    
        helm repo add stable https://charts.helm.sh/stable          ///We need to add the Helm Stable Charts for our local client

        helm repo add prometheus-community https://prometheus-community.github.io/helm-charts     ///Add Prometheus Helm repo

        kubectl create namespace prometheus            ///Create Prometheus namespace

        helm install stable prometheus-community/kube-prometheus-stack -n prometheus      ///Install Prometheus

        kubectl get pods -n prometheus          ///To check whether Prometheus is installed

        kubectl get svc -n prometheus           ///to check the services file (svc) of the Prometheus


    3--let’s expose Prometheus to the external world using LoadBalancer
    
        kubectl edit svc stable-kube-prometheus-sta-prometheus -n prometheus    ///type:LoadBalancer, change port & targetport to 9090, save and close

        kubectl get svc -n prometheus    //copy dns name of LB and browse with 9090 you will be able to see all the targets

    4--login to Grafana and add above prometheus url/dns followed by :9090 in targets and create an dashboard using id #15760

16. Now for auto trigger we can configure "webhook" so for that :

    -- go to jenkins and select configure option for your pipeline , after that select "github project" and "github hook trigger" option.

    -- go to github and select "setting" for your application repo and configure webhook with jenkins url "http://"ip_addr":8080/github-webhook/

    -- configure git with your cli and verify auto trigger using below steps:

          git config --global user.name "your.name"

          git config --global user.email "your-email-address"

          git clone https://github.com/Rojha-git/tomcat-app.git

          cd a-youtube-clone-app

          git add .

          git commit -m "test change"

          git push origin main


![Final-app](https://github.com/Rojha-git/tomcat-app/blob/main/images/tomcat-application.png)

**run "kubectl get svc" and browse dns followed by port :8080 and by :8080/webapp/ , you will be able to see our finle application that we have deployed**.

![Reg-App](

**Thankyou 
  Happy Learning**      
             
