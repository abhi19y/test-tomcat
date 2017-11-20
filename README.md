# test-tomcat
Using Ansible Playbook to install Apache,2 Tomcat instances &amp; Mysql server

# Using Ansible Playbook to install Apache,2 Tomcat instances & Mysql server

This playbook would simply install Apache2 , Mysql and 2 Apache Tomcat instances. It would get tomcat instances from my S3 bucket and also change .bashrc file for CATALINA_HOME and then deploy simple war file provided by tomcat named as "sample.war" in both tomcat instances.

## Getting Started

Clone the repository and run follwoing command.

	ansible-playbook playbook.yml

### Prerequisites


Ansible should be installed, if its not installed then use following command 

	sudo apt-get install ansible

Also git should be installed, if not installed already then use follwoing command.

	sudo apt-get install git

### Load Balancing configuration is done manually by using follwoing commands 

We would need following modules to be enabled 

	sudo a2enmod proxy
	sudo a2enmod proxy_http
	sudo a2enmod proxy_balancer
	sudo a2enmod lbmethod_byrequests

Once done restart your apache

	sudo systemctl restart apache2

After restarting we need to edit virtualhost present in /etc/apache2/sites-available/000-default.conf

	<VirtualHost *:80>
	<Proxy balancer://mycluster>
    	BalancerMember http://127.0.0.1:8080/sample # First tomcat running on 8080 port
    	BalancerMember http://127.0.0.1:8180/sample # Second Tomcat running on 8180 port
	</Proxy>

    	ProxyPreserveHost On

	    ProxyPass / balancer://mycluster/
	    ProxyPassReverse / balancer://mycluster/
	</VirtualHost>

Here we are Load Balancing Across Multiple Backend Servers which are tomcat instances.

#### Tomcat configuration is already done and stored in S3 which has public access for now.

#### scripts folder contains two scripts which would automatially run tomcat when server starts , just copy scripts to /etc/init.d/tomcat & /etc/init.d/tomcat-7
	sudo cp tomcat /etc/init.d/tomcat
	sudo cp tomcat-7 /etc/init.d/tomcat-7

#### Change its permissions and  add symlinks 
	chmod 755 /etc/init.d/tomcat
	chmod 755 /etc/init.d/tomcat-7
	update-rc.d tomcat-7 defaults
	update-rc.d tomcat defaults

