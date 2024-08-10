# Jenkins Tomcat Demo

## How to Install Tomcat
Follow below URL to deploy tomcat on Ubuntu 22.04

https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-10-on-ubuntu-20-04
```
- sudo useradd -m -d /opt/tomcat -U -s /bin/false tomcat
- sudo apt update
- sudo apt-get install openjdk-17-jdk -y
- cd /tmp
- wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.28/bin/apache-tomcat-10.1.28.tar.gz
- sudo tar xzvf apache-tomcat-10*tar.gz -C /opt/tomcat --strip-components=1
- sudo chown -R ubuntu:ubuntu /opt/tomcat/
- sudo chmod -R u+x /opt/tomcat/bin
```
Open tomcat-users.xml file and add roles & users for managing tomcat. Execute below command and details in below table. 
To exit from the nano editor press Cntrl+X and then press Y
```
sudo nano /opt/tomcat/conf/tomcat-users.xml
```
Paste in tomact.users.xml file
------------------------------------------------------------------------------------------------------------------------
```
<role rolename="manager-gui, manager-script" />
<user username="manager" password="manager_password" roles="manager-gui, manager-script" />

<role rolename="admin-gui" />
<user username="admin" password="admin_password" roles="manager-gui,admin-gui" />
```
------------------------------------------------------------------------------------------------------------------------


To remove the restriction for the Manager page, open its config file for editing and COMMENT out the Valve definition like below
```
sudo nano /opt/tomcat/webapps/manager/META-INF/context.xml
```
------------------------------------------------------------------------------------------------------------------------
...
```
<Context antiResourceLocking="false" privileged="true" >
  <CookieProcessor className="org.apache.tomcat.util.http.Rfc6265CookieProcessor"
                   sameSiteCookies="strict" />
<!--  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" /> -->
  <Manager sessionAttributeValueClassNameFilter="java\.lang\.(?:Boolean|Integer|Long|Number|String)|org\.apache\.catalina\.filters\.Csr>
</Context>
```
------------------------------------------------------------------------------------------------------------------------

Do the same thing for host manager as well
```
sudo nano /opt/tomcat/webapps/host-manager/META-INF/context.xml
```
------------------------------------------------------------------------------------------------------------------------
...
```
<Context antiResourceLocking="false" privileged="true" >
  <CookieProcessor className="org.apache.tomcat.util.http.Rfc6265CookieProcessor"
                   sameSiteCookies="strict" />
<!--  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" /> -->
  <Manager sessionAttributeValueClassNameFilter="java\.lang\.(?:Boolean|Integer|Long|Number|String)|org\.apache\.catalina\.filters\.Csr>
</Context>
```
------------------------------------------------------------------------------------------------------------------------

Create the tomcat service file
```
sudo nano /etc/systemd/system/tomcat.service
```

Paste the below contents. Make sure the variable JAVA_HOME is set correctly. 

NOTE: If you have a different Java version then you can check the Java path by running the below command path of Java is correct
- sudo update-java-alternatives -l
------------------------------------------------------------------------------------------------------------------------
```
[Unit]
Description=Tomcat
After=network.target

[Service]
Type=forking

User=ubuntu
#Group=tomcat

Environment="JAVA_HOME=/usr/lib/jvm/java-1.17.0-openjdk-amd64"
Environment="JAVA_OPTS=-Djava.security.egd=file:///dev/urandom"
Environment="CATALINA_BASE=/opt/tomcat"
Environment="CATALINA_HOME=/opt/tomcat"
Environment="CATALINA_PID=/opt/tomcat/temp/tomcat.pid"
Environment="CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC"

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh

RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
```
------------------------------------------------------------------------------------------------------------------------

Execute the below steps to start the Tomcat server
```
- sudo systemctl daemon-reload
- sudo systemctl start tomcat
- sudo systemctl status tomcat

Press q to exit the command.

- sudo ufw allow 8080
```
You can access Tomcat server on browser using URL http://your_server_ip:8080


------------------------------------------------------------------------------------------------------------------------


## Steps to Configure Jenkins

### Deploying using SCP command
- Add SSH Agent plugin
- Add  Tomcat server private key (.pem file) as Jenkins credential, (tomcat-key)
- Create a Jenkins pipeline Job
- Define a parameter as name TOMCAT_IP in the Job, as this is used in Jenkinsfile

### Deploying using Container Plugin
- Add "Deploy to Container" Plugin
- Please add tomcat-manager as a credentials in jenkins. Use the username and password define for manager while configuring tomcat above.
- PLease allow traffic from Jenkins Machine, by adding Jenkins IP address for all ports in the tomcat machine security group
- Add String parameter TOMCAT_IP to the Job

## Manual Approval
The input directive in Jenkins Pipeline is used to pause the execution of a pipeline at a specific point and wait for human interaction or approval before proceeding further. It allows you to introduce manual intervention into your automated pipeline, enabling users to review the pipeline progress and make decisions.

PLease refer the file Jenkins-Input-Slack for how to add a input statement in the pipeline and how to capture and use the values provided by User
in other stages of the pipeline. 

The submitter property of input directive controls, who can approve a build. We need to provide a valid jenkins user's UserID/Username as the 
submitter. A Admin user will be able to approve a build

To allow a user to approve a build using the input step in a Jenkins Pipeline, you need to grant the user the necessary permissions in the Jenkins permission matrix. Here are the required permissions:
```
Overall/Read: This permission allows the user to view the job and its status.
Job/Build: This permission allows the user to initiate a build.
Job/Cancel: This permission allows the user to cancel a build.
Job/Configure: This permission allows the user to configure the job, including the pipeline script.
```
