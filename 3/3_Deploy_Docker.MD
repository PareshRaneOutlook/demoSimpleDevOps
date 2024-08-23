# Installing Docker on Amazon Linux server

### Pre-requisites
1. Amazon Linux EC2 Instance

## Installation Steps

1. Install docker and start docker services
   ```sh 
   yum install docker -y
   docker --version 
   
   # start docker services
   service docker start
   service docker status
   ```
1. Create a user called dockeradmin
   ```sh
   useradd dockeradmin
   passwd dockeradmin
   ```
1. add a user to docker group to manage docker 
   ```
   usermod -aG docker dockeradmin
   ```
### Validation test
1. Create a tomcat docker container by pulling a docker image from the public docker registry
   ```sh

   docker pull tomcat

   docker run -d --name tomcat-container -p 8090:8080 tomcat:latest
   ```
2. open port 8081 -9000 inbound rules on security group of EC2 Docker Server

3. try to open http://[EC2 public ip]:8090/ (May error out due to Status 404 error)

   >> Fix issue manually
   
   docker exec -it tomcat-container /bin/bash

   ls webapps            -- no files present
   cd webapps.dist
   cp -R * ../webapps/

   try http://13.234.21.244:8090/ now

   >> fix issue with dockerfile.

   Create Dockerfile with content from "tomcat_fix_issue_Dockerfile" in this folder

   docker build -t mytomcat .

   docker run -d --name mytomcat-container -p 8090:8080 mytomcat:latest

4. Integrate Docker With Jenkins

   Create user dockeradmin
      useradd dockeradmin
      passwd dockeradmin

   Add dockeradmin to docker group
      usermod -aG docker dockeradmin

   Enable Password based login to EC2
      vi /etc/ssh/sshd_config
         
      #PasswordAuthentication no
      PasswordAuthentication yes		>>SAVE
       
	   service sshd reload

   Add user as sudoer

         
	   
  	   add following text at last line(shift + g), press escape, enter :wq

	   dockeradmin ALL=(ALL) NOPASSWD: ALL  
	   
       
	   service sshd reload
   
   Add Plugin "Publish Over SSH" to Jenkins Server under Manage Jenkins > Plugin

   Configure Publish Over SSH to Jenkins Server under Manage Jenkins > System
      Add SSH

         Name        :   dockerhost
         Hostname    :   private ip address of docker-server
         Username    :   dockeradmin

      Goto Advance and add password and test configuration

5. Jenkins Job to build and deply artefact on docker server

      Create new job and copy config from job_buildAndDeployMavenProject

      Remove Post Build Deploy war ear to Container

      Add "SSH build artefact over SSH"

         Source files      :  webapp/target/*.war
         Remove prefix     :  webapp/target
         Remote directory  :  


6. Update Dockerfile to automate deployment process

      login as root to docker server

         cd /opt
         mkdir docker
         chown -R dockeradmin:dockeradmin docker
         mv Dockerfile /opt/docker/
         chown -R dockeradmin:dockeradmin /opt/docker

      Change Jenkins Job COnfig

         Remote Directory     :  //opt//docker

      Change DockerFile and add COPY command to Dockerfile, SAVE
      
         vi Dockerfile
         
         COPY ./*.war /usr/local/tomcat/webapps

         docker build -t tomcat:v1 .

         docker run -d --name tomcatv1 -p 8090:8080 tomcat:v1

      Login using browser

         http://13.234.21.244:8090/webapp/

7. Automate build and Deployment on Container using jenkins
   
   Updated Container job created in previous step and Write following in exec section
      cd /opt/docker;
      docker build -t regapp:v1 .;
      docker run -d --name registerapp -p 8087:8080 regapp:v1

   Remove all containers and images

      docker stop containerID
      docker rm containerID
      docker image rm imageID

   Updated Container job created in previous step and Write following in "exec" section as you can't rerun this

      cd /opt/docker;
      docker build -t regapp:v1 .;
      docker stop registerapp;
      docker rm registerapp;
      docker run -d --name registerapp -p 8087:8080 regapp:v1