# 0 Pre-requitsite
	```sh
		AWS Account
		VS Code
		Download https://mobaxterm.mobatek.net/download-home-edition.html
	```
# 1 Setup Jenkins Sever
	```sh
		EC2 instance (Linux AMI 2) free tier
			If its already ready just use command 
				service jenkins start on mobxterm

		
		Open mobxterm to access ec2 instance from your portal
			>> session 
			>> specify public ip_Address of ec2 
			>> user name : ec2-user 
			>> Advance Setting > Use private key > load pem key >> Open
			>> login
			
			
			>> login as root sudo su - for installation

				cat /etc/hostname/

			
			>> Jenkins installation from https://www.jenkins.io/download/ >> got for centos /fedora / liniux (https://pkg.jenkins.io/redhat-stable/)
 				
				sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
  				sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key

			  amazon-linux-extras install epel -y
			  amazon-linux-extras install java-openjdk11 -y
			  java -version
			  yum install jenkins -y
			  
			  cat /var/lib/jenkins/secrets/initialAdminPassword	:	93b4ce93e2854f6d88fdb54e1be09e68

			  
			  service jenkins status
			  service jenkins start
			  
			  open security tcp port 8080
			  
			  create sample job just to verify everything works

	# 2 Integrate Git on Jenkins
				
				install git on Jenkins
					yum install git -y
					git --version

				install git plugin on Jenkins	

				Manage Jenkins >> Manage Plugins >>Installed >> Available >> Github

				Configure Jenkins

				Manage Jenkins >> Tools >> Setup 
				
				
				fork git repo
					create jenkins job to pull git repo(job_pullcodefromgithub)
				
				##Integrate Maven with Jenkins
	# 3 Integrate Maven on Jenkins			  
				setup Maven on Jenkins Server
						cd /opt
						wget https://dlcdn.apache.org/maven/maven-3/3.8.8/binaries/apache-maven-3.8.8-bin.tar.gz
						tar -xvzf apache-maven-3.8.8-bin.tar.gz
						mv apache-maven-3.8.8 maven
						cd bin
						./mvn -v

				setup env variable java_home, m2, m2_home
						
						cd ~
						ll -a
						nano .bash_profile
						
						
						
				In another ssh session 
						find / -name java-11*
						/usr/lib/jvm/java-11-openjdk-11.0.23.0.9-2.amzn2.0.1.x86_64
						
						file content 
						<<<<<<<<<<<<<<<
						M2_HOME=/opt/maven
						M2=/opt/maven/bin
						JAVA_HOME=/usr/lib/jvm/java-11-openjdk-11.0.23.0.9-2.amzn2.0.1.x86_64
						# User specific environment and startup programs

						PATH=$PATH:$HOME/bin:$JAVA_HOME:$M2_HOME:$M2

						>>>>>>>>>>>>
						
						source .bash_profile
						echo $PATH
						mvn -v

		
				
				Add mavem plugin to Jenkins
				Manage Jenkins >> Manage Plugins >>Installed >> Available >> maven

				configure maven and java
					Manage Jenkins >> Tools >> Setup 
						In tools add java_home and maven
	```
# 2 Setup Tomcat Sever
	```sh
	### Pre-requisites
	1. EC2 instance with Java 11
		amazon-linux-extras install java-openjdk11 -y

	### Install Apache Tomcat
	
	1. Download tomcat packages from  https://tomcat.apache.org/download-80.cgi onto /opt on EC2 instance
		cd /opt
		wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.91/bin/apache-tomcat-9.0.91.tar.gz
		tar -xvzf apache-tomcat-9.0.91.tar.gz
		mv apache-tomcat-9.0.91 tomcat
		cd tomcat/bin/
		startup.sh
		
		http://<Public_IP>:8080
		

	1. give executing permissions to startup.sh and shutdown.sh which are under bin. 
	
	chmod +x /opt/tomcat/bin/startup.sh 
	chmod +x /opt/tomcat/bin/shutdown.sh
	
	> Note: you may get below error while starting tomcat incase if you dont install Java   
	`Neither the JAVA_HOME nor the JRE_HOME environment variable is defined At least one of these environment variable is needed to run this program`
	1. create link files for tomcat startup.sh and shutdown.sh 
	
	ln -s /opt/tomcat/bin/startup.sh /usr/local/bin/tomcatup
	ln -s /opt/tomcat/bin/shutdown.sh /usr/local/bin/tomcatdown
	tomcatup
	
	#### Check point :
	access tomcat application from browser on port 8080  
	- http://<Public_IP>:8080

	Using unique ports for each application is a best practice in an environment. But tomcat and Jenkins runs on ports number 8080. Hence lets change tomcat port number to 8090. Change port number in conf/server.xml file under tomcat home
	
	cd /opt/tomcat/conf
	# update port number in the "connecter port" field in server.xml
	# restart tomcat after configuration update
	tomcatdown
	tomcatup
	



	#### Check point :
	Access tomcat application from browser on port 8090  
	- http://<Public_IP>:8090

	1. now application is accessible on port 8090. but tomcat application doesnt allow to login from browser. changing a default parameter in context.xml does address this issue
	
	#search for context.xml
	find / -name context.xml

	1. above command gives 3 context.xml files. comment (<!-- & -->) `Value ClassName` field on files which are under webapp directory. 
	After that restart tomcat services to effect these changes (comment it out)

	<!--  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
			allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" /> -->


	At the time of writing this lecture below 2 files are updated. 
	
	/opt/tomcat/webapps/host-manager/META-INF/context.xml
	/opt/tomcat/webapps/manager/META-INF/context.xml
	
	# Restart tomcat services
	tomcatdown  
	tomcatup
	
	1. Update users information in the tomcat-users.xml file
	goto tomcat home directory and Add below users to conf/tomcat-users.xml file
	
		<role rolename="manager-gui"/>
		<role rolename="manager-script"/>
		<role rolename="manager-jmx"/>
		<role rolename="manager-status"/>
		<user username="admin" password="admin" roles="manager-gui, manager-script, manager-jmx, manager-status"/>
		<user username="deployer" password="deployer" roles="manager-script"/>
		<user username="tomcat" password="s3cret" roles="manager-gui"/>
	
	1. Restart serivce and try to login to tomcat application from the browser. This time it should be Successful
	```
# 3 Setup Docker Sever
	```sh
	### Pre-requisites
	1. Amazon Linux EC2 Instance

	## Installation Steps

	1. Install docker and start docker services
	
	yum install docker -y
	docker --version 
	
	# start docker services
	service docker start
	service docker status
	
	1. Create a user called dockeradmin
	
	useradd dockeradmin
	passwd dockeradmin
	
	1. add a user to docker group to manage docker 
	
	usermod -aG docker dockeradmin
	
	### Validation test
	1. Create a tomcat docker container by pulling a docker image from the public docker registry
	
	docker run -d --name test-tomcat-server -p 8090:8080 tomcat:latest
	

	## Docker Installation on CentOS server
	##### Referance URL : https://docs.docker.com/install/linux/docker-ce/centos/#install-using-the-repository
	### Pre-requisites

	Please follow below steps to install docker CE on CentoOS server instance. For RedHat only Docker EE available 

	1. Install the required packages.

	
	sudo yum install -y yum-utils \
	device-mapper-persistent-data \
	lvm2
	
	
	1. Use the following command to set up the stable repository.
	
	
	sudo yum-config-manager \
	--add-repo \
	https://download.docker.com/linux/centos/docker-ce.repo
	```

	### INSTALLING DOCKER CE

	1. Install the latest version of Docker CE.
	
	sudo yum install docker-ce
	

	Note: If prompted to accept the GPG key, verify that the fingerprint matches 
	060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35, and if so, accept it.

	1. Start Docker.
	
	sudo systemctl start docker
	

	1. Verify that docker is installed correctly by running the hello-world image.
	
	sudo docker run hello-world
	
	```

#	4 setup Ansible Server
	```sh
	# Ansible Installation

	Ansible is an open-source automation platform. It is very, very simple to set up and yet powerful. Ansible can help you with configuration management, application deployment, task automation.

	### Pre-requisites

	1. An AWS EC2 instance (on Control node)

	### Installation steps:
	#### on Amazon EC2 instance

	1. Install python and python-pip
	
	yum install python
	yum install python-pip

	1. Install ansible using pip check for version
	
	pip install ansible
	ansible --version
	
	
	1. Create a user called ansadmin (on Control node and Managed host)  
	
	useradd ansadmin
	passwd ansadmin
	
	1. Below command grant sudo access to ansadmin user. But we strongly recommended using "visudo" command if you are aware vi or nano editor.  (on Control node and Managed host)
	
	echo "ansadmin ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
	
	
	1. Log in as a ansadmin user on master and generate ssh key (on Control node)
	
	sudo su - ansadmin
	ssh-keygen
	
	1. Copy keys onto all ansible managed hosts (on Control node)
	
	ssh-copy-id ansadmin@<target-server>
	

	1. Ansible server used to create images and store on docker registry. Hence install docker, start docker services and add ansadmin to the docker group. 
	
	yum install docker
	
	# start docker services 
	service docker start
	service docker start 
	
	# add user to docker group 
	usermod -aG docker ansadmin

	
	1. Create a directory /etc/ansible and create an inventory file called "hosts" add control node and managed hosts IP addresses to it. 
	
	### Validation test

	
	1. Run ansible command as ansadmin user it should be successful (Master)
	
	ansible all -m ping

   ```

#	5 KEKS Bootstrap Server
	```sh
	Setup Kubernetes on Amazon EKS

	You can follow same procedure in the official  AWS document [Getting started with Amazon EKS – eksctl](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html)   

	#### Pre-requisites: 
	- an EC2 Instance  (EKS Bootstrap Server)
	- Install AWSCLI latest verison  (https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
	
		aws --version
		curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
		unzip awscliv2.zip
		sudo ./aws/install

	1. Setup kubectl   (https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html)
	a. Download kubectl version latest  
	b. Grant execution permissions to kubectl executable   
	c. Move kubectl onto /usr/local/bin   
	d. Test that your kubectl installation was successful    

	
		--ignore  curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.21.2/2021-07-05/bin/linux/amd64/kubectl
	curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.30.2/2024-07-12/bin/linux/amd64/kubectl
	chmod +x ./kubectl
	mv ./kubectl /usr/local/bin 
	kubectl version

	2. Setup eksctl  ()
	a. Download and extract the latest release   
	b. Move the extracted binary to /usr/local/bin   
	c. Test that your eksclt installation was successful   

	
	curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
	sudo mv /tmp/eksctl /usr/local/bin
	eksctl version
	
		
	3. Create an IAM Role and attache it to EC2 instance    
	`Note: create IAM user with programmatic access if your bootstrap system is outside of AWS`   
	IAM user should have access to   
	IAM   
	EC2   
	CloudFormation  
	VPC
	Note: Check eksctl documentaiton for [Minimum IAM policies](https://eksctl.io/usage/minimum-iam-policies/)
	
	4. Create your cluster and nodes 

	eksctl create cluster --name cluster-name  \
	--region region-name \
	--node-type instance-type \
	--nodes-min 2 \
	--nodes-max 2 \ 
	--zones <AZ-1>,<AZ-2>
	
	example:
	eksctl create cluster --name mydemo \
	--region ap-south-1 \
	--node-type t2.small 
	
	```
