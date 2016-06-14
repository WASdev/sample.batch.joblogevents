# sample.batch.joblogevents

This sample makes use of the ***batch-1.0*** and ***batchManagement-1.0*** features of the WebSphere Liberty Profile.

It is a MDB application that demos the job log as an event functionality of the batch product. It receives batch job log event messages from all the servers in the Liberty environment that have batch events enabled and produce job logs. These messages are used to create a local job log directory structure with the actual batch job logs. 

**IMPORTANT:** This sample requires that batch job events and job logging are enabled within your liberty environment. For more information on these two topics please see the two links below.

* [A document](http://www.ibm.com/support/knowledgecenter/en/was_beta_liberty/com.ibm.websphere.wlp.nd.multiplatform.doc/ae/twlp_batch_monitoring.html) discussing how to enable batch job event publishing for Java Batch in WebSphere
* [A document](http://www.ibm.com/support/knowledgecenter/en/was_beta_liberty/com.ibm.websphere.wlp.nd.multiplatform.doc/ae/rwlp_batch_view_joblog.html) going into more detail about job logging for Java Batch in WebSphere

## Steps to set up and use this sample app

**1.) Download the pre-built WAR file or build the WAR file using the source code and maven**

**2.) Place the WAR file in the desired server's "dropins" directory**

**3.) Update the server.xml file for the chosen server to include the following:**

```xml
<!--  This is used by the MDB application activation spec to filter for topics in the topic tree. -->
<jmsTopic id="JobLogEventTopic" jndiName="jms/batch/batchJobTopic">
	<properties.wasJms topicName="batch//jobs//execution//jobLogPart//." />
</jmsTopic>

<!-- The MDB application to create job log directories from job log events-->
<jmsActivationSpec id="JobLogEventsDirCreator-1.0/JobLogEventsSubscriber">
	<properties.wasJms
		destinationRef="JobLogEventTopic" destinationType="javax.jms.Topic" />
</jmsActivationSpec>
```
__Note:__ This allows the MDB application to receive all the batch job log event messages, which contain the job log contents.

**4.) Either start or restart the updated server.**

Now the MDB application will create a directory called "JobLogEvents" under the configured server's main directory. It will contain all of the batch job log parts from all servers that have batch events enabled and produce job log parts.

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
