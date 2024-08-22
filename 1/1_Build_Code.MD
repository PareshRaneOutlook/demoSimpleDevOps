### Pre-requitsite
		AWS Account
		VS Code
		
		Download https://mobaxterm.mobatek.net/download-home-edition.html
BUILD CODE

# 1 Setup Jenkins Sever
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
					./mvn -v   (Run this from /opt/maven/bin folder)

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
			Manage Jenkins >> Manage Plugins >> Available >> maven

			configure maven and java
				Manage Jenkins >> Tools >> Setup 
					In tools add java_home and maven
			Add maven_home : /opt/maven
			
			Create new jenkins job for maven project (job_buildmavenproject)
			
			set up git repo
			
			Write appropriate goal (https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html)
			
			ll /var/lib/jenkins/workspace/job_buildmavenproject/webapp/target/
			
 SO Now FAR WE HAVE BUILT OUR Code
 




					
					
	
			  
			  
			  
			  
			  
			  
			  
			  
			  
			  

			
			
			
			
		