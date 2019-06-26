# sample.batch.joblogevents

This sample makes use of the ***batch-1.0*** and ***batchManagement-1.0*** features of WebSphere Liberty.

It is a MDB application that demos the job log as an event functionality of the batch product. It receives batch job log event messages from all the servers in the Liberty environment that have batch events enabled and produce batch job logs. These messages are used to create one local job log directory structure with the actual batch job log files as opposed to having multiple job log directories across multiple servers. 

**IMPORTANT:** This sample requires that batch job events and job logging are enabled within your liberty environment. For more information on these two topics please see the two links below.

* [A document](http://www.ibm.com/support/knowledgecenter/en/was_beta_liberty/com.ibm.websphere.wlp.nd.multiplatform.doc/ae/twlp_batch_monitoring.html) discussing how to enable batch job event publishing for Java Batch in WebSphere
* [A document](http://www.ibm.com/support/knowledgecenter/en/was_beta_liberty/com.ibm.websphere.wlp.nd.multiplatform.doc/ae/rwlp_batch_view_joblog.html) going into more detail about job logging for Java Batch in WebSphere

The steps below assume you are using two (or three) Liberty servers.  One server actually runs the batch job.  A second server hosts the MDB that captures the job log.  A third server may be configured to host the WebSphere messaging engine (or you can use MQ).  This can all be combined into fewer servers, but the separation seemed to make it clearer what is going on.

## Steps to set up and use this sample app

**1.) Download the pre-built WAR file or build the WAR file using the source code and maven**

**2.) Place the WAR file in the desired server's "dropins" directory**

**3.) If using the WebSphere integrated messaging engine, configure the wasJmsServer-1.0 feature as well as the following:**

```xml
<wasJmsEndpoint host="*" 
		  wasJmsPort="7280" 
		  wasJmsSSLPort="7290" 
		  enabled="true"> 
</wasJmsEndpoint>

<messagingEngine>
	<queue id="batchLibertyQueue" 
		forceReliability="ReliablePersistent" 
		receiveAllowed="true"/>
</messagingEngine>    
```

**4.)  Configure a server to run your batch job.  The server must also be configured to publish batch event messages, which requires the batchManagement-1.0 as well as the wasJmsClient-2.0 features. Configuration to publish to the WebSphere messaging engine described above would look like this:**

```xml
    <batchJmsEvents connectionFactoryRef="batchConnectionFactory" />
    
    <jmsConnectionFactory id="batchConnectionFactory" jndiName="jms/batch/connectionFactory">
       <properties.wasJms
       remoteServerAddress="localhost:7280:BootstrapBasicMessaging"
       />
    </jmsConnectionFactory>
```

If using MQ as the messaging engine, use properties.wmqJms with appropriate values.  The first document link above has samples.

**5.)  Configure a server to host the MDB and subscribe to batch event topic tree.  It will need to enable the wasJmsClient-2.0 feature.  If using the WebSphere messaging engine it will look like this:**

```xml
<jmsTopic id="JobLogEventTopic" jndiName="jms/batch/batchJobTopic">
	<properties.wasJms topicName="batch//jobs//execution//jobLogPart//." />
</jmsTopic>

<!-- The MDB application to create job log directories from job log events-->
<jmsActivationSpec id="JobLogEventsDirCreator-1.0/JobLogEventsSubscriber">
	<properties.wasJms
		destinationRef="JobLogEventTopic" destinationType="javax.jms.Topic" 
		remoteServerAddress="localhost:7280:BootstrapBasicMessaging"
		/>
</jmsActivationSpec>
```

__Note:__ This allows the MDB application to receive all the batch job log event messages, which contain the job log contents.

If using MQ for the messaging engine the updates will look more like this:

```xml
<jmsTopic id="JobLogEventTopic" jndiName="jms/batch/batchJobTopic">
 <properties.wmqJms
   baseTopicName=“batch/jobs/execution/jobLogPart”
   brokerDurSubQueue=“SYSTEM.JMS.D.JSRMON” />
</jmsTopic>

<jmsActivationSpec id=“JobLogEventsDirCreator-1.0/JobLogEventsSubscriber”>
     <properties.wmqJms
      destinationRef=“JobLogEventTopic”
      destinationType=“javax.jms.Topic”
      transportType=“CLIENT”
      channel=“SYSTEM.DEF.SVRCONN”
      queueManager=“MQS1"
      hostName=“wg31.washington.ibm.com”
      port=“1414" />
</jmsActivationSpec>
```

__Note:__ For a durable subscription a queue is required to hold the messages while the subscriber is not available.  That queue name must start 'SYSTEM.JMS'.  There is a default queue for this but we've created a different one here.  You'll need to change the annotations in the code to indicate you want a Durable subscription.

**6.) Start the required servers and submit your batch job.**

Now the MDB application will create a directory called "JobLogEvents" under the configured server's main directory. It will contain all of the batch job log parts from all servers that have batch events enabled and produce job log parts.

**Example directory structure created:**

![alt tag](https://github.com/WASdev/sample.batch.joblogevents/blob/master/images/jobLogDirStructure.png)

## Notice

© Copyright IBM Corporation 2016.

## License

```text
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
````
