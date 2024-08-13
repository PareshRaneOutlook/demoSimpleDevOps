
### Pre-requisites
1. EC2 instance with Java 11
	amazon-linux-extras install java-openjdk11 -y

### Install Apache Tomcat
```sh
1. Download tomcat packages from  https://tomcat.apache.org/download-80.cgi onto /opt on EC2 instance
	cd /opt
	wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.91/bin/apache-tomcat-9.0.91.tar.gz
	tar -xvzf apache-tomcat-9.0.91.tar.gz
	mv apache-tomcat-9.0.91 tomcat
	cd tomcat/bin/
	startup.sh
	
	http://<Public_IP>:8080
	
   ```
1. give executing permissions to startup.sh and shutdown.sh which are under bin. 
   ```sh
   chmod +x /opt/tomcat/bin/startup.sh 
   chmod +x /opt/tomcat/bin/shutdown.sh
   ```
   > Note: you may get below error while starting tomcat incase if you dont install Java   
   `Neither the JAVA_HOME nor the JRE_HOME environment variable is defined At least one of these environment variable is needed to run this program`
1. create link files for tomcat startup.sh and shutdown.sh 
   ```sh
   ln -s /opt/tomcat/bin/startup.sh /usr/local/bin/tomcatup
   ln -s /opt/tomcat/bin/shutdown.sh /usr/local/bin/tomcatdown
   tomcatup
   ```
  #### Check point :
access tomcat application from browser on port 8080  
 - http://<Public_IP>:8080

  Using unique ports for each application is a best practice in an environment. But tomcat and Jenkins runs on ports number 8080. Hence lets change tomcat port number to 8090. Change port number in conf/server.xml file under tomcat home
   ```sh
 cd /opt/tomcat/conf
# update port number in the "connecter port" field in server.xml
# restart tomcat after configuration update
tomcatdown
tomcatup
```



#### Check point :
Access tomcat application from browser on port 8090  
 - http://<Public_IP>:8090

1. now application is accessible on port 8090. but tomcat application doesnt allow to login from browser. changing a default parameter in context.xml does address this issue
   ```sh
   #search for context.xml
   find / -name context.xml
   ```
1. above command gives 3 context.xml files. comment (<!-- & -->) `Value ClassName` field on files which are under webapp directory. 
After that restart tomcat services to effect these changes (comment it out)

<!--  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" /> -->


At the time of writing this lecture below 2 files are updated. 
   ```sh 
   /opt/tomcat/webapps/host-manager/META-INF/context.xml
   /opt/tomcat/webapps/manager/META-INF/context.xml
   
   # Restart tomcat services
   tomcatdown  
   tomcatup
   ```
1. Update users information in the tomcat-users.xml file
goto tomcat home directory and Add below users to conf/tomcat-users.xml file
   ```sh
	<role rolename="manager-gui"/>
	<role rolename="manager-script"/>
	<role rolename="manager-jmx"/>
	<role rolename="manager-status"/>
	<user username="admin" password="admin" roles="manager-gui, manager-script, manager-jmx, manager-status"/>
	<user username="deployer" password="deployer" roles="manager-script"/>
	<user username="tomcat" password="s3cret" roles="manager-gui"/>
   ```
1. Restart serivce and try to login to tomcat application from the browser. This time it should be Successful



### Deploy tomcat plugin to Jenkins
	To change admin password
		Dashboard >> Manage Jenkins >> Jenkins’ own user database >> admin		
	
	To add tomcat deployer
		Dashboard >> Manage Jenkins >> Credentials >> System >> Global credentials (unrestricted)
		
		
Create a new Buildand Deploy Maven Project Job ()


