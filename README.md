# ConnectTech

This project contains the performance analysis of three popular NoSQL databases (Cassandra, MongoDB and HBase) using YCSB Benchmark Testing.

NoSQL databases have been extensively used on cloud systems due to their schema-free data architecture, horizontal scalability, and ability to handle large volumes of data. Large amounts of data require systems that are capable of not only to retrieve information in a very small-time frame but also to scale at the same pace as data increases. The increased popularity of data analytics, data mining, along with machine learning has guided the creation of a wide variety of NoSQL databases. One of several key advantages of NoSQL databases is the horizontal scalability, i.e. the capacity to scale efficiency as the number of machines connected to the current cluster increases. This capability potentially wastes resources due to over-provisioning. It is also necessary to be able to define the correct number of nodes needed for a given workload.


As part of the implementation, we deployed Amazon EC2 instances, installed and set up the databases along with tables that were used for insert/read/update/delete purposes. The implementation included following steps:

    1. Creating and Deploying Amazon EC2 Linux instances.
    
    2. Installing and setting up NoSQL databases in the instance.
    
    3. Create and load a workload using YCSB.
    
    4. Run the workload in the instance and compare the results.
    
    
Each instance type includes one or more instance sizes, allowing you to scale your resources to the requirements of your target workload.


We have performed experiments on General Purpose i.e. t2 instances, and Table 1 shows the instance types used for this implementation.

![Alt text](https://github.com/sinha-ashu/ConnectTech/blob/main/data/InstanceTypes.png?raw=true)


## **Step1: Creating and Deploying Amazon EC2 Linux instances**


* We can launch a Linux instance using the following  AWS Management Console as described in the following steps  
    Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/
* From the console dashboard,choose Launch instance.
* The Choose an Amazon Machine Image (AMI) page displays a list of basic configurations, called Amazon Machine Images (AMIs), that serve as templates for your instance. Select an HVM version of Amazon Linux 2.
* On the Choose an Instance Type page, you can select the hardware configuration of your instance. Select the t2.large instance type to start with.
* Choose Review and Launch to let the wizard complete the other configuration settings for you.
* On the Review Instance Launch page, under Security Groups, you'll see that the wizard created and selected a security group for you. You can use this security        group, or alternatively you can select the security group that you created when getting set up using the following steps:
* Choose Edit security groups.
* On the Configure Security Group page, ensure that Select an existing security group is selected.
* Select your security group from the list of existing security groups, and then choose Review and Launch.
* On the Review Instance Launch page, choose Launch.
* When prompted for a key pair, select Choose an existing key pair, then select the key pair that you created when getting set up.
* When you are ready, select the acknowledgement check box, and then choose Launch Instances.
* Connect to the instance using SSH – In a terminal window, use the ssh command to connect to the instance. You specify the path and file name of the private key (.pem), the username for your instance, and the public DNS name. 



## **Step 2: Installing and setting up NoSQL databases in the instance:**


We followed following steps for corresponding database setups,

#### HBase Install and Setup:

* Run below commands, To Update your Ubuntu system before starting deployment of Hadoop and HBase.

        sudo apt update
 
        sudo apt -y upgrade
 
        sudo reboot
  
* Run below commands, Install Java if it is missing on your Ubuntu 18.04 system.

        sudo add-apt-repository ppa:webupd8team/java
        
        sudo apt update
        
        sudo apt-get install openjdk-8-jdk


* Set JAVA_HOME variable.

        cat <<EOF | sudo tee /etc/profile.d/hadoop_java.sh
        export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
        export PATH=\$PATH:\$JAVA_HOME/bin
        EOF
        
        source /etc/profile.d/hadoop_java.sh

* Creating a separate user Hadoop and generate ssh keys for this user doing all tasks related to Hadoop and HBase.

        sudo adduser hadoop
        
        sudo usermod -aG sudo hadoop
        
        sudo su - hadoop
        
        ssh-keygen -t rsa
        
  
 * Add this user’s key to list of Authorized ssh keys.

        cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
        
        chmod 0600 ~/.ssh/authorized_keys


* Downloading and installing the Hadoop 3.2.1.

        RELEASE="3.2.1"
        
        wget https://www-eu.apache.org/dist/hadoop/common/hadoop-$RELEASE/hadoop-$RELEASE.tar.gz
        
        tar -xzvf hadoop-$RELEASE.tar.gz
        
        rm hadoop-$RELEASE.tar.gz
        
        sudo mv hadoop-$RELEASE/ /usr/local/hadoop
        
        
* Set HADOOP_HOME and add directory with Hadoop binaries to your $PATH.

        sudo vim  /etc/profile.d/hadoop_java.sh
        
 * Add following lines to the existing file.
 
        export HADOOP_HOME=/usr/local/hadoop
        export HADOOP_HDFS_HOME=$HADOOP_HOME
        export HADOOP_MAPRED_HOME=$HADOOP_HOME
        export YARN_HOME=$HADOOP_HOME
        export HADOOP_COMMON_HOME=$HADOOP_HOME
        export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
        export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

* set PATH variable 
        
        source /etc/profile.d/hadoop_java.sh
        
* Configuring Hadoop while adding properties to following files core-site.xml, hdfs-site.xml, mapred-site.xml and yarn-site.xml  and validate the configuration by * starting and stopping dfs and yarn.


* First edit JAVA_HOME in shell script hadoop-env.sh:

        sudo vim /usr/local/hadoop/etc/hadoop/hadoop-env.sh 
        
* Set JAVA_HOME - Line 54 as below 

        export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64

* Downloading and installing HBase 2.2.5 in Standalone mode.
* Edit configuration file hbase-site.xml for operating it in single mode.
* Start dfs, yarn and hbase to check the status.


#### MongoDB Install and Setup:

* Deploy the instance with the operating system as Ubuntu 18.04 LTS and update all the packages.
* Importing the public key used by the package management system.
        
        wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
* Creating a list file for MongoDB.
        
        echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
* Reloading and updating the local package database.
        
        sudo apt-get update
* Downloading and installing the latest stable version of MongoDB.
        
        sudo apt-get install -y mongodb-org
* Start the MongoDB process and check if it works by not giving any error message. 
        
        sudo systemctl start mongod
        sudo systemctl status mongod    
        
#### Cassandra Install and Setup:

* Deploy the instance with the operating system as Ubuntu 18.04 LTS and update all the packages.
* Download and Install Java.
* Create a new file and add the Apache repository of Cassandra to the file /etc/yum.repos.d/cassandra.repo (as the root user).
* Update the package index from sources:
* Install Cassandra using YUM. A new Linux user cassandra will get created as part of the installation. The Cassandra service will also be run as this user.
* Start the Cassandra service.
* Monitor the progress of the startup 
* Check the status of Cassandra.The status column in the output should report UN which stands for “Up/Normal”.

## **Step 3: Create and load a workload using YCSB.**

We followed following steps for corresponding database for creating and loading a workload for YCSB,

#### Benchmarking HBase with YCSB:

* Download the latest release of YCSB.
* Run below two commands in HBase shell to create a HBase table for testing
    
        n_splits = 10 

        create 'usertable', 'family', {SPLITS => (1..n_splits).map {|i| "user#{1000+i*(9999-1000)/n_splits}"}}


* Before we can actually run the workload, we need to "load" the data first. We should specify a HBase config directory(or any other directory containing  hbase-site.xml) and a table name and a column family(-cp is used to set java classpath and -p is used to set various properties).
Run below command to load the workload,

        bin/ycsb load hbase2 -P workloads/workloada -cp /HBASE-HOME-DIR/conf -p table=usertable -p columnfamily=family


#### Benchmarking MongoDB with YCSB:

* Download and setup MongoDB 18.04 version.
* Download and install Java and Maven.
        
        sudo apt-get install maven
* Download Python version 2.7 or below because YCSB doesn’t work with Python version 3.0 or above.
        
        sudo apt-get install python
* If not logged in as root user, switch to root user before installing YCSB.
        
        sudo su -
* Install and set up the YCSB benchmarking tool.
        
        curl -O --location https://github.com/brianfrankcooper/YCSB/releases/download/0.5.0/ycsb-0.5.0.tar.gz
        tar xfvz ycsb-0.5.0.tar.gz
        cd ycsb-0.5.0


#### Benchmarking Cassandra with YCSB:

* Download the latest release of YCSB.
* Create a keyspace called yscb using the following command.

        create keyspace ycsb WITH REPLICATION = {'class' : 'SimpleStrategy', 'replication_factor': 3};

* Create a table called ‘usertable’ using the following command.

        create table usertable ( y_id varchar primary key, field0 varchar, field1 varchar, field2 varchar, field3 varchar, field4 varchar, field5 varchar, field6 varchar, field7 varchar, field8 varchar, field9 varchar);

* To load the workload, run the below command.

        ./bin/ycsb load cassandra2-cql -p hosts="127.0.0.1" -P workloads/workloada -p table=usertable -p columnfamily=family


## **Step 4 : Run the workload in the instance and compare the results.**

Once the workload is loaded to databases, we need to run the YCSB tool on respective databases.

#### YCSB Run on HBase:

* Run below command,

        bin/ycsb run hbase2 -P workloads/workloada -cp /HBASE-HOME-DIR/conf -p table=usertable -p columnfamily=family


#### YCSB Run on MongoDB:

* Now to run the test, first we use the asynchronous driver to load the data:

        ./bin/ycsb load mongodb-async -s -P workloads/workloada > outputLoad.txt
* Then, run the workload:

        ./bin/ycsb run mongodb-async -s -P workloads/workloada > outputRun.txt
        
#### YCSB Run on Cassandra:

* Run the following command 

        ./bin/ycsb run cassandra2-cql -p hosts="127.0.0.1" -P workloads/workloada -p table=usertable -p columnfamily=family
        


