# Ansible Installation

Ansible is an open-source automation platform. It is very, very simple to set up and yet powerful. Ansible can help you with configuration management, application deployment, task automation.

### Pre-requisites

1. An AWS EC2 instance (on Control node)

### Installation steps:
#### on Amazon EC2 instance

   
2. Create a user called ansadmin (on Control node and Managed host)  
   ```sh
   useradd ansadmin
   passwd ansadmin
   
   visudo
   
   ansadmin ALL=(ALL) NOPASSWD: ALL  >> escape + :wq
   
   nano /etc/ssh/sshd_config
   #PasswordAuthentication no
   PasswordAuthentication yes		>>SAVE
   
   service sshd reload
   ```
   
3. Log in as a ansadmin user on master and generate ssh key (on Control node)
   ```sh 
   sudo su - ansadmin
   ssh-keygen
   ```
   
4. Install ansible2
   ```sh
	amazon-linux-extras install ansible2
	
	python --version
	ansible --version

   Alternate way using this link
   	(https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-and-upgrading-ansible-with-pip
   	after installation run $ ansible-config init --disabled > ansible.cfg
   	)
   ```


5. Add ansdamin to docker server

   ```sh
      useradd ansadmin
      passwd ansadmin

      visudo
	   
  	add following text at last line(shift + g), press escape, enter :wq

	   ansadmin ALL=(ALL) NOPASSWD: ALL  
   Password based login already enable while integrating with docker
      grep Password /etc/ssh/sshd_config

   ...

5. Add docker server as managed node on ansible server

   ```sh
   Add docker server private ip address to ansible host inventory file

      sudo vi /etc/ansible/hosts
   
   Copy ansadmin ssh keys to target ansadmin useraccount
      
      ll -la
      cd .ssh
      ssh-copy-id <docker-server-ip>

   login to docker server as ansadmin and verify

      ll -la
      cd .ssh
      ll
      date
      cat authorized_keys

   Verify key with ansible server
      cat id_rsa.pub

   Verify by issuing following command

      ansible all -m ping
   ...


7. Integrate Ansible with Jenkins

   ```sh

   Goto Jenkins portal and    Add Ansible Server under Manage Jeniks > System > Add SSH Server

      Name        : ansible-server
      Hostname    : private ip address of ansible server
      username    : ansadmin

      password of ansadmin account under advance and test configuration

   ...


9. Create Jenkins Job by copying container job we created earlier

   ```sh
   job_Copy_Artifect_Onto_Ansible

      disable poll scm
      change docker serber to ansible-server
      remove exec section

   create /opt/docker on ansible server
         cd /opt
         sudo mkdir docker
         sudo chown -R ansadmin:ansadmin docker

   ...


9. Create docker Image on Ansible logi as ansadmin

   ```sh

   Ansible server used to create images and store on docker registry. Hence install docker, start docker services and add ansadmin to the docker group.

   Check if docker is present or not
      cat /etc/group

      sudo yum install docker

      # add user to docker group 
      sudo usermod -aG docker ansadmin

      # start docker services 
      sudo service docker status
      sudo service docker start 
      

      create Dockerfile with following content

      cd /opt/docker

      vi Dockerfile

      FROM tomcat:latest
      RUN cp -R /usr/local/tomcat/webapps.dist/* /usr/local/tomcat/webapps
      COPY ./*.war /usr/local/tomcat/webapps

      docker build -t regapp:v1 .   >>>> may fail so run following command

      sudo chmod 777 /var/run/docker.sock
      docker build -t regapp:v1 .
      docker run -t --name regapp-server -p 8081:8080 regapp:v1
 
   ```
10. Create Ansible playbook to create image and container

   ```sh
   Add Ansible server private ip address to /etc/ansible/hosts


      sudo vi /etc/ansible/hosts

         [dockerhost]
         172.31.5.85

         [ansible]
         172.31.13.231

   Verify By running following (you can not do self ping or uptime)
      ansible all -a uptime

   Add ssh key for own server
      ssh-copy-id <private ip address of ansible server>


   Write Ansible Playbook with content of regapp_1.yml

      sudo vi regapp.yml

   Check and Run 
      ansible-playbook regapp.yml  --check
      ansible-playbook regapp.yml

   ```

11. Push Image to dockerhub using Ansible

   ```sh
      
   Login to docker
     
      docker login
      docker tag 3008b5d910b7 pareshrane/regapp:latest
      docker images


   ```

12. Open Jenkins job to build an image onto ansible

   ```sh

   Update regapp.yml with content of file regapp_2.yml

   Check and run

      ansible-playbook regapp.yml --check
      ansible-playbook regapp.yml


   Type following command in exec section

      ansible-playbook /opt/docker/regapp.yml

   ```

13. Create container using ansible

   ```sh

   Login to docker server
      service docker start
      docker images

      docker rm containerid
      docker image rm imageid

   create new file deploy_regapp.yml on ansible server as per content deploy_regapp_1.yml

      sudo vi deploy_regapp.yml
      ansible-playbook deploy_regapp.yml --check   
      ansible-playbook deploy_regapp.yml           >> this will fail due to access issue on docker

   Login to docker server and run following
      chmod 777 /var/run/docker.sock
   
   Run on ansible server
      ansible-playbook deploy_regapp.yml 

   Update ansible playbook to remove existing container and images as per content in deploy_regapp_2.yml
      sudo vi deploy_regapp.yml
      ansible-playbook deploy_regapp.yml --check   
      ansible-playbook deploy_regapp.yml  

   Check Docker Image modules
      https://docs.ansible.com/ansible/latest/collections/community/docker/docker_image_module.html
   ```