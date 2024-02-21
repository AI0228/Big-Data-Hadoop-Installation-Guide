# Setup Hadoop on Windows 10 machines

Consolidated instructions on how to setup and run Hadoop on Windows 10 machines. This is exactly written from [Hadoop 3.2.1 Installation on Windows 10 step by step guide](https://kontext.tech/column/hadoop/377/latest-hadoop-321-installation-on-windows-10-step-by-step-guide). Big thanks to [Raymond](https://kontext.tech/profile/u-42), the original writer. If you already have Hadoop installed and configured on your machine, you can go to the Running MapReduce Jobs section.

## Required tools
1. Java JDK - used to run the Hadoop since it's built using Java
2. 7Zip or WinRAR - unzip Hadoop binary package; anything that unzips `tar.gz`
3. CMD or Powershell - used to test environment variables and run Hadoop

## Step 1 - Download and extract Hadoop
Download Hadoop from their official website and unzip the file. We'll be using [Hadoop 3.2.1](https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-3.2.1/hadoop-3.2.1.tar.gz). Hadoop is portable so you can store it on an external hard drive. For the purpose of documentation, I will extract it to `C:/Users/Anthony/Documents/cp-master`.

If there are permission errors, run your unzipping program as administrator and unzip again.

## Step 2 - Install Hadoop native IO binary
Clone or download the [`winutils` repository](https://github.com/cdarlint/winutils) and copy the contents of `hadoop-3.2.1/bin` into the extracted location of the Hadoop binary package. In our example, it will be `C:\Users\Anthony\Documents\cp-master\hadoop-3.2.1\bin`

## Step 3 - Install Java JDK
Java JDK is required to run Hadoop, so if you haven't installed it, install it.

Oracle requires you sign up and login to download it. I suggest you find an alternative resource to download it from [for example here (JDK 8u261)](https://enos.itcollege.ee/~jpoial/allalaadimised/jdk8/). This resource might not exist forever, so Google 'jdk version download'.

Run the installation file and the default installation directory will be `C:\Program Files\Java\jdk1.8.0_261`. 

After installation, open up CMD or Powershell and confirm Java is intalled:
```
$ java -version
java version "1.8.0_261"
Java(TM) SE Runtime Environment (build 1.8.0_261-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.261-b12, mixed mode)
```

## Step 4 - Configure environment variables

Open the Start Menu and type in 'environment' and press enter. A new window with System Properties should open up. Click the `Environment Variables` button near the bottom right.

### `JAVA_HOME` environment variable
1. From step 3, find the location of where you installed Java. In this example, the default directory is `C:\Program Files\Java\jdk1.8.0_261`
2. Create a new **User variable** with the variable name as `JAVA_HOME` and the value as `C:\Program Files\Java\jdk1.8.0_261`

### `HADOOP_HOME` environment variable
1. From step 1, copy the directory you extracted the Hadoop binaries to. In this example, the directory is `C:\Users\Anthony\Documents\cp-master\hadoop-3.2.1`
2. Create a new **User variable** with the variable name as `HADOOP_HOME` and the value as `C:\Users\Anthony\Documents\cp-master\hadoop-3.2.1`

### `PATH` environment variable 
We'll now need to add the bin folders to the `PATH` environment variable.
1. Click `Path` then `Edit`
2. Click `New` on the top right
3. Add `C:\Users\Anthony\Documents\cp-master\hadoop-3.2.1\bin` 
4. Add `C:\Program Files\Java\jdk1.8.0_261\bin`

### Hadoop environment
Hadoop complains about the directory if the `JAVA_HOME` directory has spaces. In the default installation directory, `Program Files` has a space which is problematic. To fix this, open the `%HADOOP_HOME%\etc\hadoop\hadoop-env.cmd` and change the `JAVA_HOME` line to the following:
```
set JAVA_HOME=C:\PROGRA~1\Java\jdk1.8.0_261
```

After setting those environment variables, you reopen CMD or Powershell and verify that the `hadoop` command is available:
```
$ hadoop -version
java version "1.8.0_261"
Java(TM) SE Runtime Environment (build 1.8.0_261-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.261-b12, mixed mode)
```

## Step 5 - Configure Hadoop
Now we are ready to configure the **most important part** - Hadoop configurations which involves Core, YARN, MapReduce, and HDFS configurations. 

Each of the files are in `%HADOOP_HOME%\etc\hadoop`. The full path for this example is `C:\Users\Anthony\Documents\cp-master\hadoop-3.2.1\etc\hadoop`

### Configure core site
Edit `core-site.xml` and replace the `configuration` element with the following:
```
<configuration>
  <property>
    <name>fs.default.name</name>
    <value>hdfs://0.0.0.0:19000</value>
  </property>
</configuration>
```

### Configure HDFS
Create two folders, one for the namenode directory and another for the data directory. The following are the two created folders in this example:

1. `C:\Users\Anthony\Documents\cp-master\hadoop-3.2.1\data\dfs\namespace_logs`
2. `C:\Users\Anthony\Documents\cp-master\hadoop-3.2.1\data\dfs\data`

Edit `hdfs-site.xml` and replace the `configuration` element with the following:
```
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <!-- <value>file:///DIRECTORY 1 HERE</value> -->
    <value>file:///C:/Users/Anthony/Documents/cp-master/hadoop-3.2.1/data/dfs/namespace_logs</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <!-- <value>file:///DIRECTORY 2 HERE</value> -->
    <value>file:///C:/Users/Anthony/Documents/cp-master/hadoop-3.2.1/data/dfs/data</value>
  </property>
</configuration>
```

### Configure MapReduce and YARN site
Edit `mapred-site.xml` and replace the `configuration` element with the following:
```
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
  <property> 
    <name>mapreduce.application.classpath</name>
    <value>%HADOOP_HOME%/share/hadoop/mapreduce/*,%HADOOP_HOME%/share/hadoop/mapreduce/lib/*,%HADOOP_HOME%/share/hadoop/common/*,%HADOOP_HOME%/share/hadoop/common/lib/*,%HADOOP_HOME%/share/hadoop/yarn/*,%HADOOP_HOME%/share/hadoop/yarn/lib/*,%HADOOP_HOME%/share/hadoop/hdfs/*,%HADOOP_HOME%/share/hadoop/hdfs/lib/*</value>
  </property>
</configuration>
```

Edit `yarn-site.xml` and replace the `configuration` element with the following:
```
<configuration>
  <property>
    <name>yarn.resourcemanager.hostname</name>
    <value>localhost</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name>yarn.nodemanager.env-whitelist</name>
    <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
  </property>
</configuration>
```

## Step 6 - Initialize HDFS and bugfix
Run the following command and you should find the following error:
```
$ hdfs namenode -format
...
ERROR namenode.NameNode: Failed to start namenode.
...
```
To fix this, you'll need to [download the a JAR file with the fix](https://github.com/FahaoTang/big-data/blob/master/hadoop-hdfs-3.2.1.jar). Overwrite the existing `hadoop-hdfs-3.2.1.jar` in `%HADOOP_HOME%\share\hadoop\hdfs` with this new JAR file (you can make a backup of the current one before overwriting if you wish). 

## Step 7 - Start HDFS daemons
Run the following command to start HDFS daemons. When you do so, there should be two new windows that open: one for datanode and the other for namenode:
```
$ %HADOOP_HOME%\sbin\start-dfs.cmd
```

## Step 8 - Start YARN daemons
You might encounter permission issues as a normal user, so open a command line with elevated permissions. **If you have the [`yarn`](https://yarnpkg.com/) package manager, you will NOT be able to run YARN daemons since both use the `yarn` command.** To fix this, you must uninstall `yarn` package manager. 

Run the following command (with elevated permissions) to start YARN daemons. When you do so, there should be two new windows that open: one for resource manager and the other for node manager:
```
$ %HADOOP_HOME%\sbin\start-yarn.cmd
```

## Step 9 - Useful Web portals
The daemons also host websites that provide useful information about the cluster

### HDFS Namenode UI info
[http://localhost:9870/dfshealth.html#tab-overview](http://localhost:9870/dfshealth.html#tab-overview)

### HDFS Datanode UI info
[http://localhost:9864/datanode.html](http://localhost:9864/datanode.html)

### YARN resource manager UI
[http://localhost:8088](http://localhost:8088)

## Step 10 - Shutdown YARN and HDFS daemons
You can stop the daemons by running the following commands:
```
$ %HADOOP_HOME%\sbin\stop-dfs.cmd
$ %HADOOP_HOME%\sbin\stop-yarn.cmd
```

# Running MapReduce Jobs
After setting up your environment and running the HDFS and YARN daemons, we can start working on running MapReduce jobs on our local machine. We need to compile our code, produce a JAR file, move our inputs, and run a MapReduce job on Hadoop.

## Step 1 - Configure extra environment variables
As a preface, it is best to setup some extra environment variables to make running jobs from the CLI quicker and easier. You can name these environment variables anything you want, but we will name them `HADOOP_CP` and `HDFS_LOC` to not potentially conlict with other environment variables.

Open the Start Menu and type in 'environment' and press enter. A new window with System Properties should open up. Click the `Environment Variables` button near the bottom right.

### `HADOOP_CP` environment variable
This is used to compile your Java files. The backticks (eg. \``some command here`\`) do not work on Windows so we need to create a new environment variable with the results. If you need to add more packages, be sure to update the `HADOOP_CP` environment variable.

1. Open a CLI and type in `hadoop classpath`. This will produce all the locations to the Hadoop libraries required to compile code, so copy all of this
2. Create a new **User variable** with the variable name as `HADOOP_CP` and the value as the results of `hadoop classpath` command

### `HDFS_LOC` environment variable
This is used to reference the HDFS without having to constantly type the reference

1. Create a new **User variable** with the variable name as `HDFS_LOC` and the value as `hdfs://localhost:19000` 

After creating those two extra environment variables, you can check by calling the following in your CLI:
```
$ echo %HADOOP_CP%
$ echo %HDFS_LOC%
```

## Step 2 - Compiling our project
Run the following commands in your CLI with your respective `.java` files.
```
$ mkdir dist/
$ javac -cp %HADOOP_CP% <some_directory>/*.java -d dist/
```

## Step 3 - Producing a JAR file
Run the following commands to create a JAR file with the compiled classes from Step 2.
```
$ cd dist
$ jar -cvf <application_name>.jar <some_directory>/*.class
added manifest
...
```

## Step 4 - Copy our inputs to HDFS
Make sure that HDFS and YARN daemons are running. We can now copy our inputs to the HDFS using the `copyFromLocal` command and verify the contents with the `ls` command :
```
$ hadoop fs -copyFromLocal <some_directory>/input %HDFS_LOC%/input
$ hadoop fs -ls %HDFS_LOC%/input
Found X items
...
```

## Step 5 - Run our MapReduce Job
Run the following commands in the `dist` folder when we originally compiled our code:
```
$ hadoop jar <application_name>.jar <pkg_name>.<class_name> %HDFS_LOC%/input %HDFS_LOC%/output
2020-10-16 17:44:40,331 INFO client.RMProxy: Connecting to ResourceManager at localhost/127.0.0.1:8032
...
2020-10-16 17:44:43,115 INFO mapreduce.Job: Running job: job_1602881955102_0001
2020-10-16 17:44:55,439 INFO mapreduce.Job: Job job_1602881955102_0001 running in uber mode : false
2020-10-16 17:44:55,441 INFO mapreduce.Job:  map 0% reduce 0%
2020-10-16 17:45:04,685 INFO mapreduce.Job:  map 100% reduce 0%
2020-10-16 17:45:11,736 INFO mapreduce.Job:  map 100% reduce 100%
2020-10-16 17:45:11,748 INFO mapreduce.Job: Job job_1602881955102_0001 completed successfully
...
```

We can verify the contents of our output by using the `cat` command just like in shell:
```
$ hadoop fs -cat %HDFS_LOC%/output/part*
2020-10-16 18:19:50,225 INFO sasl.SaslDataTransferClient: SASL encryption trust check: localHostTrusted = false, remoteHostTrusted = false
...
```

## Step 6 - Copy outputs to our local machine
Once we are satisfied with the results, we can copy the contents to our local machine using the `copyToLocal` command:
```
$ hadoop fs -copyToLocal %HDFS_LOC%/output <some_output_directory>/
```