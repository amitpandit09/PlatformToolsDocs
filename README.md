# PlatformToolsDocs

1. Spin up EC2 instance Ubuntu Server 16.04 LTS AMI (Eligible for Free tier) with below options

   VPC:enabled
   
   Subnet:enabled
   
   Security group:
   
   Port 2182 for Zookeeper
   
   Port 5050 for mesos UI
   
   Port 8080 for maratho UI
   
   Login to instance using user : ubuntu
   
2. Install JDK

   sudo add-apt-repository ppa:webupd8team/java
	 
	 sudo apt-get update
	 
	 sudo apt-get install openjdk-8-jdk
	 
3. Add OpenPGP key for Mesos packages from Ubuntu’s keyserver. If the below command fails try the other command.

   command 1 : sudo apt-key adv --keyserver keyserver.ubuntu.com --recv E56151F
	 
	 command 2 : sudo apt-key adv --keyserver keyserver.ubuntu.com --recv DF7D54CBE56151BF
	 
4. Next step is to add Mesos repository by forming correct URL for our Ubuntu release

   As we are using Ubuntu 16.04, the codename for it is xenial. If you’re using a different version, check the codename using the following command
	 
	 >> lsb_release -cs
	 
	 >> echo "deb http://repos.mesosphere.com/ubuntu xenial main" >> /etc/apt/sources.list.d/mesosphere.list
	 
5. Execute apt-get update to update package indexes.

   >> sudo apt-get update
	 
6. Now we will install Mesos. Zookeeper binaries included with Mesos will also get installed. Execute the following command:

   >> apt-get -y install mesos
	 
   CONFIGURE MESOS:
	 
	 a. Get the hostname using command >> hostname -f
	 
	 b. Navigate to /etc/zookeeper/conf directory. Edit the zoo.cfg file as shown below. Uncomment server.1 and replace zookeeper1 with your hostname
	 
	 c.Edit myid file inside /etc/zookeeper/conf directory. Put integer 1 as the value in file myid. This integer is the server id for each zookeeper instance inside the cluster. It should be unique and shouldn’t be repeated in zookeeper cluster. It ranges from 1 to 255:
	 
	 >> echo -n "1" > myid
	 
	 d. Navigate to /etc/mesos directory and edit the zk file to point to the zookeeper instance. As it is a single machine, localhost will work fine, but we will still provide instance hostname as given below. Do the same for all mesos-master in cluster (if any). Internal IP can also be used instead of complete hostnames. In case of multiple zookeepers, this property can be set as:
	 
	 zk://10.102.5.183:2181,10.102.5.123:2181,10.102.5.150:2181/mesos
	 
	 e. Switch to /etc/mesos-master directory and edit the quorum file with an integer. A replicated group of servers in the same application is called a quorum, and in replicated mode, all servers in the quorum have the same configurations.
	 Note: The quorum should be set in a way that 50 percent of the master members should be present to make decisions. Ideally, this number should be greater than the number of masters divided by two. Since we are running single node, we set it to 1
	 
	 f. Next, we want to configure the hostname and IP address for our mesos-master. Create two files, ip and hostname, inside /etc/mesos-master directory with appropriate values.
	 Note: If the mesos is deployed in private subnet, use private IP for both files ip and hostname. But if deployed in public subnet for POC, use Public IP in hostname and private IP in ip file. Public IP will allow you to access Sandbox and view logs.
	 
	 
7. Now all the configurations are in place. We can now start mesos-master and mesos-slave. Use the following commands to start required services.

   >> service zookeeper start
	 
	 >> service mesos-master start
	 
	 >> service mesos-slave start
	 
	 Once all the services are started, make sure they’re running without any errors. The following should be the desired output on shell:
	 
	 >> jps
	 
	 Once all the services are up and running, open your browser and browse PUBLIC_IP:5050 to view Mesos UI.
	 
	 
8. Now unless, or until, there are no frameworks created/running in agent, nothing will show up in Mesos UI. We can run any simple shell commands using mesos-execute as shown below

9. Create a simple HelloWorld.jar that we will give to mesos-master to run on slave.

  a. Create directory demo at any accessible location. Preferably in home directory:
	
	b. Create a file HelloWorld.java inside demo directory using any editor of your choice (vi, vim or nano) and paste the following contents, then save and exit:
	
	   public class HelloWorld {
		 public static void main(String[] args) {
		 System.out.println("Hello, World!");
		 System.out.println("This will run on mesos-slave");
		 }
 		}
		
	
	c. Run the following command to compile the java file which will create a class file:
	
	>>javac HelloWorld.java
	
	d. Create a MANIFEST.MF file with the following contents:
	Manifest-Version: 1.0
	
	Main-Class: HelloWorld 
	
	e. Run the following command to create an executable jar file using java file, class file and MANIFEST.MF:
	
	>> jar cmf MANIFEST.MF HelloWorld.jar HelloWorld.class HelloWorld.java
	
	f.  Now try running the generated jar using the command shown below:
	
	>>mesos-execute --master=<HOSTNAME>:5050 --name="running-jar" --command="java -jar /location/of/jar/HelloWorld.jar"
	
	
	Check the mesos console
	 
	 
