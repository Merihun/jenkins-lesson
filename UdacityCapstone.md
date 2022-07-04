
suggested reading: https://medium.com/@sabbirhossain_70520/capstone-project-cloud-devops-udacity-25d0fd72833e
Jenkins pipeline for the rolling update using AWS EKS
I have noticed that many people are getting confused about the Capstone project. Here I am giving you a rough idea about the project.
Don’t be confused about this project. It’s really an easy project if you understand it properly. I am writing a few steps. If you follow those steps, you will be able to understand and finish it properly.
Necessary scripts and files:
CloudFormation Script. You can use your own CloudFormation scripts or else you can do it using AWS Management Console.
Jenkinsfile, Dockerfile.
Your app to deploy
Deployment and Service script for Kubernetes.
You can add your own files and scripts. Those are the necessary files you have to include for this project.
Steps
Install necessary packages and setup Jenkins
Run an EC2 instance. For free tire account use t2.micro. You can use any instance type, you want to run. Connect your EC2 instance using SSH. After that install Jenkins in the EC2 instance. To install run:
sudo apt-get update
sudo apt install -y default-jdk
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt-get update
sudo apt-get install jenkins
For information check this link: https://www.jenkins.io/doc/book/installing/#debianubuntu
Start the Jenkins server using sudo systemctl start jenkins command. Install the necessary plugins in Jenkins for implementing the pipeline. Such as Blue Ocean, AWS plugins, docker plugins, etc.
Install Docker in EC2 instance.
Run: sudo apt install docker.io
For more information check this link: https://phoenixnap.com/kb/how-to-install-docker-on-ubuntu-18-04
Start the docker service.
Run: sudo systemctl start docker
Note: You need to add a user to the docker group. Otherwise, you will get an error. To add a user run sudo usermod -a -G docker <username>
Add credentials for AWS and docker in Jenkins.
Create AWS EKS Cluster and Node group
Now you need to create an AWS EKS Cluster and node group. You can use your own CloudFormation script. Otherwise, you can use AWS CLI to create AWS EKS Cluster and node group. To create an AWS EKS Cluster and node group, you need to install aws-cli, eksctl, and kubectl in your EC2 instance, you have created earlier. Follow this link to install: https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html. After installing, run this command:
eksctl create cluster — name cluster — version 1.16 — nodegroup-name standard-workers — node-type t2.medium — nodes 3 — nodes-min 1 — nodes-max 4 — node-ami auto — region us-west-2
This command will automatically create two CloudFormation scripts for the EKS cluster and node group. It will create 3 EC2 instances as nodes. You can check in EC2 Dashboard in AWS Management Console. Like this:
  <img width="1073" alt="image" src="https://user-images.githubusercontent.com/26862785/177210504-5c8fb98f-4f55-4127-8873-7d38dc3b2fcd.png">
  
  Creating service and deployment
You need to create a script for deployment. You can use either yml or json. If you are confusing about deployment script then follow this link: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
Create a service to expose the app.
A Kubernetes Service is an abstraction that defines a logical set of Pods running somewhere in your cluster, that all provide the same functionality. When created, each Service is assigned a unique IP address (also called clusterIP). This address is tied to the lifespan of the Service, and will not change while the Service is alive. Pods can be configured to talk to the Service, and know that communication to the Service will be automatically load-balanced out to some pod that is a member of the Service
You can use a Load Balancer as a service. For more information follow this link: https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/
Create a kubectl configuration file to store the info of your cluster to interact later. Run:
aws eks — region us-west-2 update-kubeconfig — name <your cluster name>
kubectl config use-context <your cluster ARN> 
kubectl config use-context command is used to switch between clusters. After your clusters, users, and contexts are defined in one or more configuration files, you can quickly switch between clusters.
For more information follow this link: https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/
Create the rolling deployment pipeline
Create a stage to lint
Create a stage to build a docker image
Create a stage to push the docker image
Create a stage to deploy.
Your pipeline will look like this:
  
  ```
  stage(‘Linting') {
}
stage('Build docker image') {
}
stage(Push docker image') {
}
stage('Deploy container') {
}
  ```
  You need to include kubectl apply -f deployment.ymlcommand in the deploy stage. It creates and updates resources in a cluster by running this command.
Run the pipeline. Run kubectl get deployments to check if the Deployment was created. Run kubectl get services/<image name>. You will get an Endpoint ID. You will get the External IP also. Use that link and copy in the browser search bar with the appropriate port you have exposed to see that your app is working correctly.
Now you need to check whether your rolling update is working or not. Update your image (change something in your app and you can use the version for this) and again deploy it to the cluster. This time use this command.
  
  ``` kubectl set image deployments/<appname> <appname>=<image>:<tag> ```
  
  Then run kubectl rollout status deployments/<image>. You will get a message like a deployment “name” successfully rolled out. If you get that message then your rolling update is working correctly.
  
  <img width="1044" alt="image" src="https://user-images.githubusercontent.com/26862785/177210650-bd875920-604b-4ec0-a22f-17ac2b096eeb.png">


  
