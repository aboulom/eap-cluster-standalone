Configuring a JBoss Cluster in Standalone mode as a Service with mod_cluster in RHEL 
====================================================================================
Author: Allen Boulom  
Level: Beginner  
Summary: This README demonstrates how to configure a JBoss cluster in standalone mode as a 
service with mod\_cluster in Red Hat Enterprise Linux.  
Target Product: JBoss AS, Apache mod\_cluster  
Source: https://github.com/aboulom/eap-cluster-domain

Overview
--------
Clustering allows for high availability by making your application available on secondary
servers when the primary instance is down or it lets you scale up or out by increasing
the server density on the host, or by adding servers on other hosts. It can even help to
increase performance with effective load balancing such as mod_cluster between servers based 
on their respective hardware. 

Step 1: JDK Installation And Verification
-----------------------------------------

Before you install JBoss AS, you must first install a JDK. Any JDK can be used, such as Sun
JDK, OpenJDK, IBM JDK, or JRocket etc. We chose Open JDK 7 for this tutorial.

1. Installing OpenJDK:

	Issue the following command to install the JDK:

			$ yum install java-1.7.0-openjdk-devel
		
2. Confirming The Install:

	Issue the following command to confirm the proper version of the JDK is on your classpath:

			$ java -version
		
Step 2: Download JBoss And The Installation Procedure 
-----------------------------------------------------

1. Downloading JBoss AS 7.1.1.Final:

			$ wget wget http://download.jboss.org/jbossas/7.1/jboss-as-7.1.1.Final/jboss-as-7.1.1.Final.zip
		
2. Installing JBoss AS 7.1.1.Final:

			$ unzip jboss-as-7.1.1.Final.zip -d /usr/share
		
Alternatively, any directory can be chosen for the JBoss 7 installation.

Step 3: Add A JBoss Management User 
--------------------------------

Now that JBoss AS 7 is installed, we need to make sure that we create a user with the appropriate 
privileges. It is never a good idea to run JBoss as root for various reasons.

1. Create The New User:

	We create a new user called jboss by issuing the following command:

			$ adduser jboss 
		
Alternatively, any username can be used. However, the username must be specified in the jboss-as.conf file.

Step 4: Create The JBoss AS 7.1.1 Standalone Service  
---------------------------------------------------- 

1. Create The /etc/jboss-as directory: 

			$ mkdir /etc/jboss-as
		
2. Copy the modified service configuration file to the /etc/jboss-as directory:

			$ cp /usr/share/jboss-as-7.1.1.Final/bin/init.d/jboss-as.conf /etc/jboss-as
		
3. Copy the service startup script to the /etc/init.d directory, and give it execute permissions: 
   
			$ cp /usr/share/jboss-as-7.1.1.Final/bin/init.d/jboss-as-standalone.sh /etc/init.d
			$ chmod +x /etc/init.d/jboss-as-standalone.sh 
		
Step 5: Setup The Server To Listen On All Interfaces  
---------------------------------------------------- 

Once the appropriate JBoss service is created, we will configure the server to listen on all 
interfaces for the management and public interfaces.

Edit The Default standalone.xml File:

	1. Edit the standalone.xml file in your favorite text editor.

	2. Next, update the following section:

		From: 
				<interface name=”management”>
				<inet-address value=”${jboss.bind.address:127.0.0.1}/>
				</interface>
				<interface name=”public”>
				<inet-address value=”${jboss.bind.address:127.0.0.1}”/>
				</interface>

		To:
				<interface name=”management”>
				<inet-address value=”${jboss.bind.address:0.0.0.0}”/>
				</interface>
				<interface name=”public”>
				<inet-address value=”${jboss.bind.address:0.0.0.0}”/>
				</interface>

NOTE: By default, JBoss 7 will only bind to localhost. This does not allow any remote access 
to your jboss server. For our amazon aws installation, we define the jboss.bind.address property 
as 0.0.0.0 and jboss.bin.address.management property to 0.0.0.0 as well. This allows us to 
access the remote JBoss amazon instance over the internet. We could have also defined the 
hostname of the ami or the ip address. However, unless an elastic ip is used, this value 
can change. This is why we opted for 0.0.0.0.

Step 6: Activate and Start The JBoss AS Standalone Service 
------------------------------------------------------

1. Add the new jboss-eap-rhel.sh service to list of automatically started services using the chkconfig service management command:

			$ chkconfig --add jboss-as-standalone.sh
		
2. Test that the service has been installed correctly by using one of the following commands.

		For Red Hat Enterprise Linux 6:

			$ service jboss-as-standalone.sh start
			
		For Red Hat Enterprise Linux 7:

			$ service jboss-as-standalone start
			
The service will start. If you get an error, check the error logs and make sure that the options in the configuration file are set correctly.

3. To make the service start automatically when the Red Hat Enterprise Linux server starts, run the following command:

			$ chkconfig jboss-as-standalone.sh on
			
Step 7: Removing the JBoss AS Service in RHEL  
---------------------------------------------

1. If the service is running, open a terminal and stop the service with one of the following commands.

		For Red Hat Enterprise Linux 6:

			$ service jboss-as-standalone.sh stop
			
		For Red Hat Enterprise Linux 7:

			$ service jboss-as-standalone stop
			
2. Remove JBoss EAP from the list of services:

			$ chkconfig --del jboss-as-standalone.sh
			
3. Delete the service configuration file and startup script:

			$ rm /etc/init.d/jboss-eap-rhel.sh
			$ rm /etc/jboss-as/jboss-as.conf