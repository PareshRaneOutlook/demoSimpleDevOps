# Setup Kubernetes on Amazon EKS

You can follow same procedure in the official  AWS document [Getting started with Amazon EKS – eksctl](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html)   

#### Pre-requisites: 
  - an EC2 Instance  (EKS Bootstrap Server)
  - Install AWSCLI latest verison  (https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
 
	aws --version
	curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
	unzip awscliv2.zip
	sudo ./aws/install

   cd ~
	ll -a
	nano .bash_profile

   << Inside file set following>>

   AWS_HOME=/usr/local/bin/aws
   PATH=$PATH:$HOME/bin:$AWS_HOME

   <<Now run following commands >>

	source .bash_profile
	echo $PATH
   aws --version

1. Setup kubectl   (https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html)
   a. Download kubectl version latest  
   b. Grant execution permissions to kubectl executable   
   c. Move kubectl onto /usr/local/bin   
   d. Test that your kubectl installation was successful    

   ```sh 
    --ignore  curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.21.2/2021-07-05/bin/linux/amd64/kubectl
   curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.30.2/2024-07-12/bin/linux/amd64/kubectl
   chmod +x ./kubectl
   mv ./kubectl /usr/local/bin 
   kubectl version
   ```
2. Setup eksctl  ()
   a. Download and extract the latest release   
   b. Move the extracted binary to /usr/local/bin   
   c. Test that your eksclt installation was successful   

   ```sh
   curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
   sudo mv /tmp/eksctl /usr/local/bin
   eksctl version
   ```
     
3. Create an IAM Role and attache it to EC2 instance    
   `Note: create IAM user with programmatic access if your bootstrap system is outside of AWS`   
   IAM user should have access to   
   IAM   
   EC2   
   CloudFormation  
   VPC
   Note: Check eksctl documentaiton for [Minimum IAM policies](https://eksctl.io/usage/minimum-iam-policies/)
   
4. Create your cluster and nodes 
   ```sh
eksctl create cluster --name cluster-name  \
   --region region-name \
   --node-type instance-type \
   --nodes-min 2 \
   --nodes-max 2 \ 
   --zones <AZ-1>,<AZ-2>
   
   example:
   eksctl create cluster --name mydemo --region ap-south-1 --node-type t2.small 
   
5. Create Manifest file and apply
   ```sh
   Please use regapp-deploy.yml and regapp-service.yml

   kubectl apply -f regapp-deploy.yml

   kubectl get all

   kubectl get pod -o wide

   kubectl apply -f regapp-service.yml

   kubectl get all

   kubectl describe service/pareshrane-service


   ```


6. Access your webapp
   ```sh
   [get loadbalancer dns name] :8080/webapp

   kubectl get pod
   kubectl delete pod  

   >>it will create one more pod

   kubectl delete -f regapp-service.yml

   kubectl delete -f regapp-deploy.yml

   ```


7. Integrate Kubernetes BS server with Ansible
   ```sh
   >> Add ansadmin user on ekssever

   useradd ansadmin
   passwd ansadmin
   
   visudo
   
   ansadmin ALL=(ALL) NOPASSWD: ALL  
   
   >> escape + :wq
   
   nano /etc/ssh/sshd_config
   #PasswordAuthentication no
   PasswordAuthentication yes		>>SAVE
   
   service sshd reload

   >> Login to  ansible server 

      sudo su - ansadmin

      cd /opt/docker

      rename regapp.yml and deploy_regapp.yml

      mv regapp.yml create_image_regapp.yml
      mv deploy_regapp.yml docker_deployment.yml

      >> create host fine under /opt/docker

      vi host
      >> Write following content

      [kubernetes]
      172.31.46.253

      [ansible]
      172.31.13.231

      >> save file

      ssh-copy-id [ip address of eks bs server]

      >> Verify login

      ssh '[ip address of eks server]' e.g. ssh '172.31.46.253'

      ansible all -a uptime

      >> login to eks server and setup password for root user
      passwd root




   ```


8. Ansible Playbook for deploy and service file
   ```sh
      >> Create kube_deploy.yml file as per content give in this file
      ssh-copy-id root@172.31.46.253


      ansible-playbook -i /opt/docker/hosts kube_deploy.yml --check
      ansible-playbook -i /opt/docker/hosts kube_deploy.yml


   ```

9. Create Jenkins job to run Ansible Playbook to launch deploy and service file
   ```sh


      >> Create freestyle CD job to run ansible playbook on kubernetes deployment; we just need  Build Step "Send build artifacts over SSH" and add following to exec section rest sfileds should be empty
      ansible-playbook -i /opt/docker/hosts /opt/docker/kube_deploy.yml

      >> Cleanup eksserver

      kubectl get all      

      kubectl delete deployment.apps/pareshrane-regapp
      kubectl delete service/pareshrane-service


      

      >> Create job from "Copy_Artefatcs_onto_Ansible" as CI job
         please remove deploy docker and change "regapp.yml" to "create_image_regapp.yml"

      >> open CI job and add post build action to "Other Project" to integrate CI and CD job
         Select regapp_CD_job

     

   ```

   
10. Cleanup Kubernetes cluster
   ```sh
      >> delete deployment
      kubectl delete deployment.apps/pareshrane-regapp
      kubectl delete service/pareshrane-service

      >> delete cluster
      kubectl get nodes
      eksctl delete cluster mydemo --region ap-south-1


   ```