# Setting up Flume for an ADLS sink
 
The project I was working on some time back was generating lots of log events which had to be filtered and later fed to different frameworks. And it made sense to push these log events to ADLS since other frameworks were already linked to an ADLS account. After analyzing, Apache Flume seemed to be the best fit and setting up Flume was the next step. But the problem was it doesn't have an ADLS sink, the closest it had was HDFS. It later came to me though that ADLS sink is not required since it's easy to set it up using HDFS sink. So, have written this article to focus on setting up ADLS sink in Apache Flume using HDFS sink.
 
To push log events to ADLS, first step is to setup sink in Flume flume-conf.properties which can be found Flume_Home\conf directory. Sample conf file is pasted below for reference.
 
# Naming the components on the current agent 
 
adlagent.sources = SeqSource   
adlagent.channels = MemChannel 
adlagent.sinks = HDFS 
 
# Describing/Configuring the source 
adlagent.sources.SeqSource.type = seq
  
# Describing/Configuring the sink
adlagent.sinks.HDFS.type = hdfs 
adlagent.sinks.HDFS.hdfs.path = adl://ccountname.azuredatalakestore.net/flume/
adlagent.sinks.HDFS.hdfs.filePrefix = log 
adlagent.sinks.HDFS.hdfs.rollInterval = 0
adlagent.sinks.HDFS.hdfs.rollCount = 10000
adlagent.sinks.HDFS.hdfs.fileType = DataStream 
 
 
# Describing/Configuring the channel 
adlagent.channels.MemChannel.type = memory 
adlagent.channels.MemChannel.capacity = 1000 
adlagent.channels.MemChannel.transactionCapacity = 100 
 
# Binding the source and sink to the channel 
adlagent.sources.SeqSource.channels = MemChannel
adlagent.sinks.HDFS.channel = MemChannel 
 
For simplicity, SeqSource source and Memory channel is used. Sink if you notice is HDFS and all properties for it is also similar except one. Property adlagent.sinks.HDFS.hdfs.path is starting from adl:// instead of hdfs:// in Hadoop world. This tells HDFS to write in ADL account rather than in HDFS. Part following adl:// is your fully qualified domain name (FQDN) of the ADLS account. It will be something like accountname.azuredatalakestore.net. For logs to be written in specific directory, one must append the directory path here e.g.: /flume/, otherwise events will be written in root directory. So, the complete path becomes adl://ccountname.azuredatalakestore.net/flume/. We are done with the Flume configuration part.
 
Trying to run it at this stage will fail as HDFS natively doesn't understand authentication and communication details for ADL scheme. For that purpose, Hadoop configuration file namely conf-site.xml has to used. Please note that for simplicity we will assume that Hadoop is not installed on system since it's not required for writing to ADLS. A sample of conf-site.xml is attached below for reference.
 
<configuration>
 
  <property>
    <name>dfs.adls.oauth2.access.token.provider.type</name>
    <value>ClientCredential</value>
  </property>
  
  <property>
    <name>dfs.adls.oauth2.refresh.url</name>
    <value>--adls account refresh url--</value>
  </property>
  
  <property>
    <name>dfs.adls.oauth2.client.id</name>
    <value>--adls account client id--</value>
  </property>
  
  <property>
    <name>dfs.adls.oauth2.credential</name>
    <value>--adls account client secret--</value>
  </property>
  
    <property>
      <name>fs.adl.impl</name>
      <value>org.apache.hadoop.fs.adl.AdlFileSystem</value>
  </property>
  <property>
      <name>fs.AbstractFileSystem.adl.impl</name>
      <value>org.apache.hadoop.fs.adl.Adl</value>
  </property>  
 
 
</configuration>
 
 
This simple core-site.xml contains configurations related only to ADLS. This file has to be placed in Flume_Home/conf/ directory so that it’s available in Flume classpath. Alternatively, it can be placed in any directory but make sure to include that directory in Flume classpath which can be configured in flume-env.ps1 or flume-env.sh (found in Flume_Home/conf/).
 
We have successfully configured the Flume and HDFS but there is one more step required, which is including the required Hadoop and Azure jars in Flume_Home\lib\ directory. There are namely 4 jars required to support this named as follows:
1.	hadoop-auth-2.8.1.jar
2.	hadoop-common-2.8.1.jar
3.	hadoop-azure-datalake-2.8.1
4.	azure-data-lake-store-sdk-2.2.3
 
Be free to try the latest stable versions available but make sure to check the compatibility. For me these mentioned specific versions worked like charm.
 
That's it, we are all set to run flume. You can run this command to try it out at Flume_Home directory.
bin\flume-ng agent -n adlagent -c conf -f conf/flume-conf.properties
 
To summarize these are the steps we followed
1.	Configured flume-conf.properties and set the HDFS path to ADL one
2.	Added core-site.xml in Flume conf directory with ADLS specific configurations
3.	Included few Hadoop and Azure related jars in Flume lib directory
 
 
Now go and check the ALDS directories already.

