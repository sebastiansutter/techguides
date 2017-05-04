---
title: 1. Starting the environment
toc: true
sidebar: sfdemo
permalink: /sfdemo_step1.html
summary: 
---

## Objectives
1. Deploy the SkyTap-based environment.
2. Configure MQSFB and start all required services.
3. Configure the App Connect Pro orchestration and redeploy it.

## 1.1 Deploy SkyTap environment
1. Log in to [cloud.skytap.com](http://cloud.skytap.com).

2. Under **Environments** -> **Templates** -> **Company**, search for `Cloud Integration for Salesforce Demo v1.0` or click [here](https://cloud.skytap.com/templates/1043585).

3. Click **New Environment** to deploy and **Run VMs** ![Run VMs](images/sfdemo_step1_runVMs.png) to start. 

## 1.2 Configure the Runtimes VM

1. Launch the **Runtimes** desktop by clicking on the VM.

2. If necessary, log in with userid `student` and password `Passw0rd!`.  

   **Note**: This is the sudo password for any steps required below.

3. Right-click the Desktop and run **Open Terminal Here**.

4. Update the MQSFB configuration to use your Salesforce.com account, in accordance with [Knowldge Center](https://www.ibm.com/support/knowledgecenter/en/SSFKSJ_9.0.0/com.ibm.mq.con.doc/q129310_.htm).

  **Note**: You may find these instructions easier to follow if you convert your Salesforce.com UI from the Salesforce Lightning Experience to the Salesforce Classic Experience.  To do so, click on your avatar icon in the top-right corner and select **Switch to Salesforce Classic**.
  
  a. Run Step 3 from the [Knowldge Center](https://www.ibm.com/support/knowledgecenter/en/SSFKSJ_9.0.0/com.ibm.mq.con.doc/q129310_.htm): _Create a self-signed security certificate in Salesforce._  
  
  **Note**: Run this from inside the Runtimes VM to avoid having to transfer the JKS file.
  
  b. Run Step 4 from the [Knowldge Center](https://www.ibm.com/support/knowledgecenter/en/SSFKSJ_9.0.0/com.ibm.mq.con.doc/q129310_.htm): _Use the IBM Key Management GUI to open the keystore you exported from Salesforce and populate the signer certificates._
  
  c. Run Step 7 from the [Knowldge Center](https://www.ibm.com/support/knowledgecenter/en/SSFKSJ_9.0.0/com.ibm.mq.con.doc/q129310_.htm): _Create a configuration file with connection and security paramters for IBM MQ, Salesforce, and the IBM MQ Bridge to Salesforce behavior._  
  
  Use the following commands:
  
  ```
  sudo cp /opt/mqm/mqsf/samp/new_config.cfg /opt/mqm/mqsf/samp/new_config.cfg.bak
  sudo /opt/mqm/bin/runmqsfb -o /opt/mqm/mqsf/samp/new_config.cfg
  sudo chmod 640 /opt/mqm/mqsf/samp/new_config.cfg
  ```
  Use the following values:
  
  ```
Connection to Queue Manager
---------------------------
Queue Manager or JNDI CF   : []QM1
MQ Base Topic              : []/sf
MQ Channel                 : []SYSTEM.DEF.SVRCONN
MQ Conname                 : []
MQ CCDT URL                : []
JNDI implementation class  : [com.sun.jndi.fscontext.RefFSContextFactory]
JNDI provider URL          : []
MQ Userid                  : []
MQ Password                : []
```
  ```
Connection to Salesforce
------------------------
Salesforce Userid (reqd)   : [] **
Salesforce Password (reqd) : [] **
Security Token (reqd)      : [] **
Login Endpoint             : [https://login.salesforce.com]
Consumer Key               : []
Consumer Secret            : []
```
  ```
Certificate stores for TLS connections
--------------------------------------
Personal keystore for TLS certificates : [] **
Keystore password          : []
Trusted store for signer certificates : []
Trusted store password     : []
Use TLS for MQ connection  : [N]
```
  ```
Behaviour of bridge program
---------------------------
PushTopic Names            : []ShipmentUpgradeRequests
Platform Event Names       : []
MQ Monitoring Frequency    : [30]
At-least-once delivery? (Y/N) : [Y]
Publish control data with the payload? (Y/N) : [N]
Delay before starting to process events : [0]
Runtime logfile for copy of stdout/stderr : []/var/mqm/qmgrs/QM1/errors/mqsfb.out
Done.
```
  
5. If MQ is already running, you will need to restart the mqsfb service.  It is configured to start automatically with the queue manager QM1, so it's easiest to simply stop the queue manager.  Unfortunately, mqsfb does not shut down automatically, so we need to invoke the following:

   ```
   /opt/mqm/bin/endmqm QM1
   ps -ef | grep mqsfb | grep -v grep | awk '{print $2}' | xargs sudo kill -9
   ```

   If MQ was not running (or now that we stopped it), invoke:

   ```
   /opt/mqm/bin/strmqm QM1
   /opt/mqm/bin/strmqweb
   ```

6. If App Connect Pro is not running, invoke:

   ```
   docker start appconnect
   ```

7. If MQSubAndPutApp is not running, invoke:

   ```
   /opt/mqm/java/jre64/jre/bin/java -Djava.library.path=/opt/mqm/java/lib64 -cp /opt/mqm/java/lib/jms.jar:/opt/mqm/java/lib/com.ibm.mqjms.jar:/home/student/Downloads/JMSRouter.jar com.ibm.mq.test.MQSubAndPutApp QM1 /sf/topic/ShipmentUpgradeRequests SF.SHIPMENTUPGRADEREQUESTS.ACP.QUEUE
```

## 1.3 Configure the Studio VM

1. Return to the SkyTap Environment console, and launch the **Studio** desktop by clicking on the VM.

2. If necessary, log in with userid `Adminstrator` and password `passw0rd`.

3. Launch **Chrome**, and click the **App Connect WMC** bookmark.

4. Login with userid `admin` and password `!n0r1t5@C`

5. From the **Dashboard**, if `CIforSFDemo` is _Running_, click **Stop** (**Cancel Jobs**), and **Undeploy**.

6. Launch **App Connect Studio**, and open the Project file `C:\Student\Projects\CIforSFDemo\CIforSFDemo.sp3`

7. In the right-hand navigation, under **Endpoints**, open **Salesforce**.

8. Update the userid and password to match your credentials.

  **Note**: Password is the concatenation of your password and your security token.  For example, if your password is `passw0rd` and your security token is `cGFzc3cwcmQ=` then you would enter `passw0rdcGFzc3cwcmQ=`.
  
9. **Save** and **publish** the project using userid `admin` and password `!n0r1t5@C`.

10. Return to the **App Connect WMC** in **Chrome**, and from the **Dashboard**, start the `CIforSFDemo` project.


