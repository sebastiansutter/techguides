---
title: FastStart 2019 Lab - CIP
toc: false
sidebar: labs_sidebar
folder: pots/cipdemo
permalink: /cip-demo-lab.html
summary: Deeper dive into the Cloud Integration Platform
applies_to: [developer,administrator,consumer]
---

# Mastering the Cloud Integration Platform
===============

## Overview

During the CIP Bootcamp, you had the opportunity to get some hands on with the CIP Platform.  This exercise here is to provide you some more hands on with the platform and build your skills with the platform with the goal of creating a demo environment everyone is able to build and use on their own.

This lab guide will lay out the scenario for you, along with the requirements of what you are to complete.  This guide is different than other labs, such that it will (for the most part) not be a step by step walkthrough of what to complete, rather will give you a list of requirements to complete, and it is up to you and your team to deliver on those.

This lab will be a team effort.  It is highly suggested that you form your teams of folks who can cover the entire I&D Portfolio - especially: App Connect Enterprise (using Toolkit), API Connect, Messaging (MQ and ES).  Its helpful also to have someone who knows `kubectl` and `docker` commands and has some background with ICP.

Lab Overview
-------------------------------------------

You will be building a demo environment that supports the "AcmeMart" demo scenario.  There are several components that make up this demo, but the key areas in scope for this lab are as follows:

* 	Two App Connect Enterprise flows that interact with IBM Cloud based APIs
*  AcmeMart Node.js based APIs
*  On-premises based MQ and ES based assets
*  API Facades for RESTful assets to be created in API Connect

All assets created are to be implemented on the given CIP environment provided for you.  All pre-requisites and utilities should be on the environments provided.  If there are other utilities and tools you would like to use to support you during this exercise, you may do so at your own risk.


Lab Environment Overview
-------------------------------------------

You have been provided a pre-installed environment of the Cloud Integration Platform with the base charts for CIP already deployed and configured.  This includes specifically:

* ICP Main Portal running on `https://mycluster.icp:8443`
* CIP Platform Navigator running on: `https://10.0.0.5/icip1-navigator1`

You should be able to access all portals from the Platform Navigator, but if you need to access the URL directly, they are provided below:

* App Connect Enterprise Manager Portal running on `https://ace.10.0.0.5.nip.io/ace-ace`
* API Connect API Manager Portal on `https://mgmt.10.0.0.5.nip.io/manager`
* MQ Portal on `https://10.0.0.5:31694/ibmmq/console/`
* Event Streams on `https://10.0.0.5:32208/gettingstarted?integration=1.0.0`

**Note**: there are some known certificate issues with the various portals on this environment. These are fairly easy to workaround, and will be fixed in a later release of this demo environment.

2. The environment you are using is the same environment used with the CIP Bootcamp.  It consists of 9 different nodes.  8 of which are ICP Nodes and one is a developer image that you will be using to access the ICP User Interfaces.  You can access this VM directly using the Skytap interface to use the X-Windows based components. The biggest difference with this environment vs what you used at the bootcamp is that all of the base CIP Charts are already installed and configured.

3. **VERY IMPORTANT** It is very important you do not suspend your lab environment.  We have seen cases when the environment goes into suspend mode, the Rook-Ceph shared storage gets corrupted.  Also it is a good practice that you shut down ICP before powering down your enviroment.  A script has been included to handle that for you that will be explained in the coming sections.  When you power down your environment, you can safely use the `power off` option as the shared storage can interfere with the normal graceful shutdown method.  So far using power off hasn't caused any problems that we are aware of.  If you find that your API Connect environments are not coming up properly, it suggested that you execute the `./icpStopStart.sh stop` script (you may need to execute it more than once).  When it stops, go ahead and execute the `./icpStopStart.sh start`.  As indicated previously, this may take ~30 minutes or so to come up.

3. You are also able to SSH into your environment.  You can find out the exact address to SSH to in your Skytap Environment window under `networking` and `published services`.  The Published Services set up for you will be for the SSH Port (22) under the `CIP Master` node.  The published service will look something like `services-uscentral.skytap.com:10000`.  Your environment will have a different port on it.  You can then SSH to to the machine from your local machine.

4. Additionally, you can access the machine direct via the Skytap UI.  This is functional, but can be cumbersome to work with as its not easy to copy and paste into and out of Skytap, especially for the non X-Windows based environments.

5. Password-less SSH has also been enabled between the Master node and the other nodes in the environment.  **note** a table with the environment configuration is provided below.  Credentials for each machine are `root`/`Passw0rd!`.

| VM        | Hostname  | IP Address | # of CPU | RAM    | Disk Space (LOCAL) | Shared Storage | Additional Notes |
|-----------|-----------|------------|----------|--------|--------------------|----------------|------------------|
| Master    | master    | 10.0.0.1   | 12       | 32 GB  | 400 GB             | N/A            |hostname mycluster.icp|
| Proxy     | proxy     | 10.0.0.5   | 4        | 8 GB   | 20 GB              | N/A            |                  |
| Worker 1  | worker-1  | 10.0.0.2   | 8        | 16 GB  | 400 GB             | Monitor  | Ceph Master      |
| Worker 2  | worker-2  | 10.0.0.3   | 8        | 16 GB  | 400 GB             | 			  | 		           |
| Worker 3  | worker-3  | 10.0.0.4   | 8        | 16 GB  | 400 GB             | 			  | 		            |
| Worker 4  | worker-4  | 10.0.0.7   | 8        | 16 GB  | 400 GB             |                |                  |
| Worker 5  | worker-5  | 10.0.0.8   | 8        | 16 GB  | 400 GB             | Ceph OSD1               |                  |
| Worker 6  | worker-6  | 10.0.0.9   | 8        | 16 GB  | 400 GB             | Ceph OSD2               |                  |
| Developer | developer | 10.0.0.6   |          |        |                    |                |                  |


6. If it is not done already, power up your Environment.  It could take about 5 minutes for all nodes to start up.  The master node takes the longest to come up, so if you can see the login prompt from the Skytap UI on the master node, then you are good to go.

7. SSH into the Master Node or use the Skytap UI.  In the home directory of root (`/root`) there is a script called `icpStopStart.sh`.  Run this script by executing `./icpStopStart.sh start` Note that it takes around 30 minutes for the ICP Services to come up completely.

8. You can tell if the start of ICP is complete by checking using one of these two methods:
	- Click on the Developer Machine, and it will take you directly to the Developer Machine running X-Windows.  Should you need to Authenticate, you can use the credentials of `student`/`Passw0rd!`. Bring up the Firefox browser.  Navigate to the main ICP UI by going to `https://mycluster.icp:8443`.  The credentials again are `admin/admin`.  If you can log in and see the main ICP Dashboard, you are good to go.
	- Alternatively, you can SSH to the Master node.  Execute a `cloudctl login` from the command line.  If all there services are up, it will prompt you for credentials (use `admin`/`admin`) and setup your kubernetes environment.
8. The best place to do your Kubernetes CLI work is from the Master node.  Again, Before you can execute any of the `kubectl` commands you will need to execute a `cloudctl login`.
9. Next step is to find The Platform Navigator UI can be use to create and manage instances of all of the components that make up the Cloud Integration Platform.
10. You can access the Platform Navigator using the browser on the developer machine.  The URL for the navigator was set up in this environment as: `https://10.0.0.5/icip1-navigator1`.  You might be asked to authenticate into ICP again, but once you do that you should now see the Platform Navigator page.
11. The Platform Navigator is designed for you to easily keep track of your integration toolset.  Here you can see all of the various APIC, Event Streams, MQ and ACE instances you have running.  You can also add new instances using the Platform Navigator.


### Key Concepts - Troubleshooting / Recovery

There are times where things may not be going right, so your best bet is to use `kubectl` commands to uncover more information about what is going on.  If you post queries in Slack about trouble with the lab, we will be asking you to execute a series of these, depending upon what you were doing, and what error occured.  A list of useful commands are provided below:

 - `kubectl get pods -n <some-namespace>` This command shows all of the pods in a given name space.  Here you will see if any pods are up, down, errored or otherwise in transition.
 - `kubectl describe pods <some-pod> -n <some-namespace>` This will provide verbose information about a given pod.  You can use the `describe` command for other objects.
 - `kubectl logs <some-object> -n <some-namespace>` This command will work with other objects as well.  You can also tail the logs by using the `-f` switch at the end.

 Logging is also done inside of ICP using the ELK stack.  You can access the logging inside of ICP and use Elastic Search commands to drill into things.  For example, if you were to bring up Kibana and enter in a search like `kubernetes.namespace:<your namespace>`.  Using this along with the `kubectl` commands gives you a lot of power to dig into root cause.


Lab Requirements
-------------------------------------------

## ACE Integration Assets

1. Your integration assets are found in this repository: `https://github.com/ibm-cloudintegration/techguides`.  You can find the specific files you need in the `/techguides/pages/cipdemo` directory.
    >**hint:** clone this on your Developer machine so you don't have to copy the files over.
2. Below is a description of each of the files in archive that you should take note of (disregard the others).

| File                           | Description                                                                                      |
|--------------------------------|--------------------------------------------------------------------------------------------------|
| faststartflows.zip             | ACE Project Interchange export of integration flows|
| inventoryproject.generated.bar | generated bar file for the Inventory API.  You will be deploying this as is into the environment|
| orderproject.generated.bar     | original bar file for order API. Disregard this, you will be generating a new bar file to deploy|
| generateSecret.sh     | a script file that generates an ICP Secret, for use when deploying BAR files|
| serverconf.yaml     | skeleton file, used by generateSecret.sh|
| setdbparms.txt     | parameter file, used by generateSecret.sh|
| truststorePassword.txt     | parameter file, used by generateSecret.sh|

  >Keep the project interchange zip file handy, you will be loading that up into the toolkit in a later section.


## AcmeMart Microservices

You will need to download the AcmeMart microservices and deploy the containers on ICP.


1. Access the `master` node via SSH session.  Make a new directory in the root home directory, call it whatever you like.  Change directories into that.
2. Once in that new directory, clone this repository here on the `master` node:  `https://github.com/asimX/FS-CIP-Microservices`.  The reason we do this on the master node because we need to load the docker images into ICP.  You might need to authenticate using your github.com account.  If you don't have one, you can sign up for your free one here at `github.com`
3. Go into the FS-CIP-Microservices directory.
3. Also, now is a good time to make sure you are logged into your ICP environment, thus:
  - Execute `sudo cloudctl login`.
  - Provide the password for student: **Passw0rd!**.
  - Ensure that the API Endpoint **https://mycluster.icp:8443** is specified. If a different one is specified, execute `sudo cloudctl logout` and try again.
  - Provide the CIP userid: **admin** with  password **admin**
  - Set the namespace context to **default**.

4. Execute docker login into the ICP.
   >`docker login mycluster.icp:8500`
   >**username:** admin
   >**password:** admin

5. Execute this command:  `docker build . -t acmemartutilityapi`.  The image and its dependencies will be downloaded.
6. In the ICP UI, starting from the top left hamburger icon select `Manage` -> `Namespaces`.  Create a new namespace and call it `acmemartapi`.  Using the `ibm-anyuid-hostpath-psp` security policy is fine for this one.
7. Tag your new image by executing this command:  `docker tag acmemartutilityapi mycluster.icp:8500/acmemartapi/acmemartutilityapi:v1.0.0`
8. Push the docker image out to your ICP instance by issuing this command `docker push mycluster.icp:8500/acmemartapi/acmemartutilityapi:v1.0.0`.
10. Next step is to deploy the microservices into ICP.  Create a new Deployment in ICP using the UI via Hamburger Icon in top left. Go to `Workloads -> Deployments`.  Click `+Create Deployment`.
11. In the `General` tab.  Give it a name of `acmemart`.  Select the new namespace created previous via the dropdown (`acmemartapi`). Leave Replicas at `1`.
12. Go to `Container Settings`.  Set the name to `acmemartutility`.  Set the `Image` value to `mycluster.icp:8500/acmemartapi/acmemartutilityapi:v1.0.0`

	![](./images/cipdemo/deployment2.gif)

12. In the same tab, scroll down to where the TCP ports are.  We need to open up a few ports on this image such that it can interact with all components.  Set the ports to the following

	- TCP - 3000
	- TCP - 9093
	- TCP - 443

13. Click `Create`.  The new pod should be created very quickly.  You click on your release and view the results in the UI.
    >**Note** you will not see a `Launch` button like you see in the screenshot until you complete the Service in the next section.

	![](./images/cipdemo/deployment_done.gif)

14. Once you see the Pod up, the next step is to bind a Network service that can be associated with the Pod.
15. From the Hamburger menu on top left go to `Network Services` -> `Services`.
16. Create a New Service.  Call it `acmemartapp`.
17. Select the proper namespace - `acmemartapi`
18. Under `Type` select from the dropdown `NodePort`

	![](./images/cipdemo/service3.gif)

19. Under the `Labels` tab set the Label to `app` and value to `acmemart`.
20. Under the `Ports` tab. Create 3 ports per the chart below:

    | Protocol  | Name         | Port | Target Port |
    |-----------|--------------|------|-------------|
    | TCP       | app          | 3000 | 3000        |
    | TCP       | eventstreams | 9093 | 9093        |
    | TCP       | kafka        | 443  | 443         |


21. Ports should look like the following when done:

	![](./images/cipdemo/serviceports.gif)

22. Under the `Selectors` tab.  Set the `Selector` to `app` and the Value to `acmemart`.
23. Click Create and the service should create quickly.
24. You will know the Service was generated properly when you return back to your deployment, and you see the `Launch` button in the upper right hand corner.

	![](./images/cipdemo/service_deployment_done_launch.gif)

25. Click the Launch button and from the dropdown, select the `app` button and it should launch the main portal for the AcmeMartUtils. If so, then your microservices for the demo has deployed successfully.
26. The AcmeMart microservices comes with its own swagger based developer documentation.  From the main App screen, click on the `Developer Docs`.

	![](./images/cipdemo/acmemart.gif)

27. There are 3 categories of APIs here.  Click on the `Utility` apis.  From the list Select, the `GET /Utilities/ping` option
28. Find the `try it out` button.  Click it.  It should return back the date/time.

	![](./images/cipdemo/pingtest.gif)

## Prepare Event Streams Details

In this section you will prepare for the integration of Event Streams and ACE.

Start by working with Event Streams, to create a topic and to define and capture connection information:
1. Navigate to the Event Streams dashboard:
  - Open a browser session to the Cloud Integration Platform home: `https://mycluster.icp/integration/`
  - Under `Messaging`, select the `es` link, to open the Event Streams Dashboard.
  - (If you see an error screen similar to that shown below, then simply select the `Open es` link in the middle.)

		![](./images/cipdemo/open_es_error.JPG)

  - Provide or accept the credentials (Username **admin** Password **admin**) , then `Log In`.
1. Select the `Topics` tab.
1. Select `Create Topic`, to start creating a new topic:
  - For the `Topic name`, we recommend **NewOrder**. This is the topic to which, in this lab, a message will be published by ACE, each time a new order is created. If any application wants to subscribe to this message then they must be told what this value is.
  - Leave the `Partitions` as **1**. (For scalability in Production, you might specify **3** or more.)
  - You can specify whatever `Message Retention` you like; we recommend leaving it as **A week**.
  - The `Replicas` setting is used for High Availability; we recommend you leave it to default.
1. Select `Connect to this cluster`, to pull in the window that allows you to create and copy configuration information.
1. Make a note of the value of the `bootstrap server`. Make sure you note it correctly, because you must specify this exactly in the ACE configuration later.
1. Use the correct button to download the `PEM Certificate`. Store it in `/home/student/Downloads`.
1. Use the section on the right to create an API key:
  - Choose an application name. This is used, within Event Streams, to report connections and activity - but it is not used further in this Lab session. You could specify **ACE-Connection**, or **ordersFlow**, for example.
  - Specify `Produce, consume and create topics`. Although in this lab session we ask you only to produce messages, specifying this allows you to extend the lab and do more later, if you want. In a Production system you could use this to restrict access.
  - Select the slider, to specify `All topics`. Although in this lab session we ask you only to use the topic you created above,  specifying `All topics` allows you to extend the lab and do more later, if you want. In a Production system you could use this to restrict access.
  - Ensure that the slider specifying `All` consumer groups is enabled. In a Production system you could use this to restrict access.
  - After selecting `Generate API key`, make sure you copy and/or download the API key.
  - Don't forget to select `Create a new API key`, to make it all happen.


You will now use the connection information to prepare the ACE Integration Server for deployment.

Firstly, you will generate a "Secret" object. This is a Kubernetes construct that lets you store and manage sensitive information, such as passwords and certificates. In this lab, a Secret is used to store connectivity information for connecting securely to Event Streams, within an ACE Helm Release (aka Integration Server).

(More information on Secrets can be seen here:
https://kubernetes.io/docs/concepts/configuration/secret/ )
1. Copy the PEM file you downloaded earlier to the correct directory.
  - On the Developer Machine, open a "Files" session.
  - Navigate to _/home/student/Downloads_.
  - Copy the PEM certificate file that you downloaded from Event Streams earlier (probably called **es-cert.pem**) to _/home/student/generateScript_.
  - Rename this PEM certificate file to the following: **truststore-escert.crt** (yes, this means that it will no longer be a PEM file). Note: that this name structure must be of the form **truststore-ALIASNAME.crt**, where ALIASNAME will be generated as the alias for the certificate. Note down the ALIASNAME **escert**, because you will need to specify this later when deploying a BAR file.
1. Edit the configuration files:
  - On the Developer Machine, open a "Terminal" session. Note that you will be signed in as _student_.
  - Navigate to _/home/student/generateScript_. This directory is already populated with the _generateSecret.sh_ tool that will generate the secret, and with the text files that form the input into that tool. This directory now also has your renamed PEM file (**truststore-ALIASNAME.crt**) in it, which also forms input into the tool.
  - Use either `gedit setdbparms.txt` or `vi setdbparms.txt` to edit the _setdbparms.txt_ file.
  - Replace the characters `<over-write with API Key>` with the API key that you saved or downloaded from Event Streams  earlier. Note that this must be done accurately, otherwise the connection from ACE to Event Streams will not work.
  - `Save` your changes and exit the editor.
1. Sign into the CIP namespace and run the tool to generate the Secret.
  - In the Terminal session, execute `sudo cloudctl login`.
  - Provide the password for student: **Passw0rd!**.
  - Ensure that the API Endpoint **https://mycluster.icp:8443** is specified. If a different one is specified, execute `sudo cloudctl logout` and try again.
  - Provide the CIP userid: **admin** with  password **admin**
  - Set the namespace context to **ace**. (We must set this now, because the Secret we are about to generate must be in namespace **ace**.)
  - Execute `./generateSecret.sh orders-secret`. This script takes in the various configuration files (including the PEM certificate and the API key) and then uses _kubectl_ to generate the Secret with name **orders-secret**. You should see a success message `secret/orders-secret created`.
1. Check that the secret has been created, using the UI.
  - In a browser session, navigate to the ICP Portal: https://mycluster.icp:8443.
	- Using the hamburger menu, navigate to `Configuration` -> `Secrets`.
	- Confirm that Secret **orders-secret** has been created, and click it to open and check that it is is namespace **ace**.


## Make changes within ACE and deploy the flows


>> Hugh: to be completed.

## Modify the Order ACE Flow

1. Import the project interchange provided in the `faststartflows.zip` file.  You can find this file in the `/home/student/techguides/pages/cipdemo` folder
>> Hugh: will we need to import this file ? Will they not already exist ?

1. On the Developer Machine, open a "Terminal" session. Note that you will be signed in as _student_, and placed in directory _/home/student_.
1. Start the Ace Toolkit by executing `./ace-v11.0.0.3/ace toolkit`.
Note that the ACE Toolkit may start as a tiny window on the screen. Use the cursor to grab the corner of this screen and expand it.

 ![](./images/cipdemo/odd-toolkit-start.JPG)

1. Drill down `orders` -> `Resources` -> `Subflows` and open `New_Order.subflow`. This subflow forms the body of the work that ACE will do when a new order is created and the orders API is called.

You will now modify this subflow, by adding three operations (nodes) following  the App Connect REST operation. The first  will strip the HTTP Headers, the second  will put to an MQ queue, and the third  will publish the message to a topic in EventStreams.

Here is what the flow will look like. Detailed instructions follow.

 ![](./images/cipdemo/orders_subflow_canvas.JPG)

1. From the ACE palette, drag and drop an `HTTPHeader` node, a `KafkaProducer` node and an `MQOutput` node. Position them and wire them up as shown in the screenshot above.
1. Select the `Http Header` node. Configure its properties thus:
  - On the `HTTPInputHeader`, set it to **Delete HTTP Header**. This is because before we do any further work with this message, we must remove this header.

 ![](./images/cipdemo/ace2.png)

2. Select the `KafkaProducer` node. Configure its properties thus:
  - Enter the topic name, matching what you defined within the Event Streams configuration earlier - we recommended **NewOrder**. Note: in this instance you are hard-coding this topic name; typically it will be parameterised (for ACE specialists: the parameterisation uses _LocalEnvironment.Destination.Kafka.Output.topicName_).
  - Leave Acks as **0**. This specifies the number of acknowledgements to request from the server before the publication request is sent. **0** is equivalent to similar to 'fire and forget'; **1** waits for a single acknowledgement; **All** waits for acknowledgements from all replicas of the topic (providing the strongest available guarantee that the message was received).
  - Change the Timeout to 5 secs (so that if it fails, you will only have to wait 5 seconds before you see the failure)
  - For Security Protocol specify **SASL_SSL**. This is because Event Streams requires this.
  - For SSL protocol specify **TLSv1.2**. This is because Event Streams requires this.


  ![](./images/cipdemo/ace2-2.png)


  ![](./images/cipdemo/ace2-3.png)

5. Select the `MQOutput` node. Configure its properties thus:
  - For Connection select `Local queue manager`. This is because the container running ACE will also include a Queue Manager, to which we will connect locally.
  - For Destination queue manager name specify **acemqserver** (case sensitive). This is the name of the local Queue Manager. Note: in this instance we are hard-coding this Queue Manager name; typically it will be parameterised (using an MQ Policy).

  ![](./images/cipdemo/ace3.png)

  - For Queue Name specify **NEWORDER.MQ**. This is the queue onto which the message will be put. Note: in this instance we are hard-coding this queue name; typically it will be parameterised (for ACE specialists: this parameterisation uses _LocalEnvironment.Destination.MQ.DestinationData.queueName_).
8. `Save` your flow.
9. Create a BAR (Broker Archive) file.
  - Call it **orders**.
	- In the `Prepare` tab, ensure that the REST API called **orders**  is selected.
	- Select `Build and Save`.

## Connecting ACE to a remote MQ on ICP
>> Comment from Hugh: I think we do not need to do this section. It was in the original lab because connection to ES was going to be via MQ. But in our lab, connection to ES doesn't use MQ. So no need to define this connection to a remote QMgr.

1. Open MQ Console on `mq` Helm Repositories
2. Click `mq-console-https 31694/TCP`
3. Click Queue Manager name: `QMGR.DEV`
4. Click  Properties
5. Find `Communication Properties` `CHLAUTH` option and Select `DISABLE` and `Save` & `Close`

  ![](./images/cipdemo/ace3-1.png)

6. Click Add Widget
	- Select Queues to add on MQ Console
	- Click on sign (+) to Create a local Queue: `NEWORDER.ES`
7. Add a new Widget
	- Select `Channel` to add on MQ Console
	- Create a channel using `Server Connection` on channels window: `ACE.TO.ES`

  ![](./images/cipdemo/ace3-3.png)

  ![](./images/cipdemo/ace3-4.png)

## Work with MQ authorization
>> Comment from Hugh: I think we do not need to do this section. It was in the original lab because connection to ES was going to be via MQ. But in our lab, connection to ES doesn't use MQ. So no need to define this security/authorisation for the remote QMgr.

1. Open a terminal window on Developer machine.
1. Sign into the relevant ICP namespace:
  - In the Terminal session, execute `sudo cloudctl login`.
  - Provide the password for student: **Passw0rd!**.
  - Ensure that the API Endpoint **https://mycluster.icp:8443** is specified. If a different one is specified, execute `sudo cloudctl logout` and try again.
  - Provide the CIP userid: **admin** with  password **admin**
  - Set the namespace context to **acemq**. (You must choose this namespace, because the kubectl work you are about to do is for this namespace.)
1. Start a bash shell to the mq pod:
  - Execute `kubectl get pods`
  - Find mq pod
  - Execute `kubectl exec -it mq-ibm-mq-0 -- /bin/bash`. You will be root user on mq server on ICP .

2. Configure security in order to allow ACE send a message to QMGR.DEV (remote MQ) .
	- Within the bash shell, create **aceuser** as part of the **mqm** group:
 	- `sudo useradd -m aceuser`
 	- `sudo usermod -a -G mqm aceuser`
	- Execute `runmqsc QMGR.DEV` to open the interactive MQ Commandline environment.
 	- Type `SET CHLAUTH(ACE.TO.ES) TYPE(BLOCKUSER) ACTION(REPLACE) USERLIST('nobody') `
 	- Type `ALTER AUTHINFO(SYSTEM.DEFAULT.AUTHINFO.IDPWOS) AUTHTYPE(IDPWOS) CHCKCLNT(OPTIONAL)`
 	- Type `ALTER QMGR CHLAUTH(DISABLED)`
 	- Type `Refresh Security`
 	- Type `end` to exit from runmqsc.

	1. Execute `exit` to terminate the bash shell.


## Deploy the bar files

In this section you will deploy the BAR Files to the CIP environment. You will deploy each one separately, and each will create an IntegrationServer in the ACE environment.

Deployment of a BAR file includes the creation of the Integration Server in which it will run, and is achieved by creating a Helm Chart. This Helm Chart includes all the details of the Integration Server, as well as the BAR file itself. In a DevOps environment, this is deployed programmatically. In a Development environment, such as this, the typical way to deploy a BAR File (and create the Integration Server) is from the ACE Dashboard.

>_**Note:**_ When deploying, you will create a unique hostname for each BAR file; this is referred to as the `NodePort IP` setting when configuring the Helm Release. For example: **orders.10.0.0.5.nip.io** and **inventory.10.0.0.5.nip.io**. The **10.0.0.5.nip.io** portion specifies that an on-demand service (**nip.io**) is used to route to **10.0.0.5** (the IP address of the ICP proxy node). The whole value of `NodePort` must be unique; it points to the HTTP listener for the specific Integration Server.

1. Use one of the following methods to get to the ACE Dashboard.
  - Either go directly by opening a browser session to **https://mycluster.icp/ace-ace1**
  - Or start with the ICP Portal **https://mycluster.icp:8443**. Choose `Workloads` -> `Helm Releases`. Find the `ace-ace` release and launch the `webui` from there.
  - Or start with the Platform Navigator: **https://10.0.0.5/icip1-navigator1**, and select `ace1`. Note: if this appears to fail as shown in the following screenshot, simply select `ace` on the failure screen and it should work.

	  ![](./images/cipdemo/open_ace_error.jpeg)


2. On the ACE Dashboard, make sure you are on the Servers tab.

3. First, deploy the `inventory` flow - which you have not changed.
  - From the ACE Dashboard, select `Create` to start the process.

  - Select the BAR file `inventoryproject.generated.bar` and continue.
  - You will be presented by the `Content URL`, and the `namespace` **ace**.  The `Content URL` defines the location, in ICP terms, of where the BAR file is.
	 - Copy the contents of the `Content URL` to the clipboard, because you will need it shortly.
	 - The namespace is being proposed by ACE; you should make a mental note of it.
    - Select `Configure` to continue.
  - ACE now selects the correct Helm Chart from the Catalog, and opens the ICP configuration pages. At the bottom, select `Configure` to continue.
  - For the `Helm Chart name`, we recommend **inventory**. This name will be used in many of the ICP artefacts, so a meaningful name is good. This name will also be used as the default for some of the later properties (for example the name of the Integration Server).
  - For the `namespace`, select **ace**, which you made a mental note of earlier.
  - Ignore the `NodePort` and select `Advanced`. (You will complete the `NodePort` shortly.)
  - Into the `Content Server URL` field, paste the contents of the `Content URL` from the clipboard. This vital piece links the BAR file to this Helm Chart.
  - For the `Image pull secret` specify **sa-ace**. This Secret was created when the Cloud Pack for Integration was installed, and it contains the credentials for ICP to access its private docker repository.
  - For the `NodePort`, we recommend **inventory.10.0.0.5.nip.io**. We recommend that the first part (**inventory** in this case) is identical to the `Integration Server name`, because there is a 1 to 1 relationship here.
  - Lower down, you could also specify the `Integration Server name`. However, we recommend that you leave this blank, so that the Helm Chart name (**inventory** in this case) is used.
  - Leave the remaining settings as defaults and then click `Install` at the bottom.
  - Your Helm Chart will now install. You can view the progress of the install via `Helm Releases` (as prompted on the screen) or via _kubectl_ on a command line.

3. You should now return to the ACE Dashboard and confirm that the Integration Server has been correctly deployed. (You should wait a few seconds to a minute, for the deployment to succeed fully.)

3. Next, deploy the new `orders` flow, which you have changed. The process to deploy this is the same as `Inventory`, with some extra configuration.
  - From the ACE Dashboard, select `Create` to start the process.
  - This time use the BAR file `orders.bar`.
  - Remember to copy the contents of the `Content URL`.
  - For this `Helm Chart name`, we recommend **orders**.
  - As before, ignore the `NodePort` and select `Advanced`.
  - As before, paste into the `Content Server URL` field.
  - As before, the `Image pull secret` is  **sa-ace**.
  - For the `NodePort`, we recommend **orders.10.0.0.5.nip.io**.
  - For the `Secret name` specify **orders-secret** as prepared earlier. This Secret adds sensitive configuration information (such as API key and a certificate) to this Integration Server, so that it can communicate with Event Streams.
  - For the `Certificate alias name`, specify **escert**. This is used by the Integration Server when it connects to Event Streams. You defined the value **escert** earlier, when you created the Secret.
  - For the `Queue manager settings` (**_Warning_**: these are case-sensitive):
    - `Queue manager name` is **acemqserver**. This must match the name that ICP gave by default to the associated queue manager (we could have changed it, but have not done so).
    - Under the heading `MQSC file for Queue Manager:`, enter **DEFINE QLOCAL(NEWORDER.MQ)**. This will create the specified MQ queue, using all defaults.

  - Leave the remaining settings as defaults and then click `Install` at the bottom.

4. You should now return to the ACE Dashboard and confirm that the Integration Server has been correctly deployed. (You should wait a few seconds to a minute, for the deployment to succeed fully.)

Now test each flow, using cURL,  before moving on.

1. Use the following value as input for the inventory API `key` query parameter : `AJ1-05.jpg`
8. Once you have confirmed the functionality for both `order` and `inventory` message flows, export the swagger for each from the ACE Management Portal.  Save it to the file system on the developer machine, you will be using this in the next section
9. Once deployed, your ACE Management UI should display both `inventory` and `order` APIs running

 ![](./images/cipdemo/appconnect.gif)

10. If you Drill into the `order` API you should see something that resembles the following (yours will be a little different)

 ![](./images/cipdemo/appconn_order_api.gif)

11. And here is `inventory`:

 ![](./images/cipdemo/appconn_inventory_api.gif)

12. Test both APIs using `cURL`.  The input for the `orders` flow is the `order.json` file that was pulled down from the `techguides` github pull.

## Review MQ Portion
To review that the MQ portion is working properly, perform  the following inside a Terminal Window on Developer Machine (signed in as student).

**Note:** ACE created the Queue Manager and there is no MQ Console available.
1. Sign into the relevant ICP namespace:
  - In the Terminal session, execute `sudo cloudctl login`.
  - Provide the password for student: **Passw0rd!**.
  - Ensure that the API Endpoint **https://mycluster.icp:8443** is specified. If a different one is specified, execute `sudo cloudctl logout` and try again.
  - Provide the CIP userid: **admin** with  password **admin**
  - Set the namespace context to **acemq**. (You must choose this namespace, because the kubectl work you are about to do is for this namespace.)
1. Execute  `kubectl get pods`
1. Find the **acemqserver** pod and copy the full name to the clipboard
1. Execute  `kubectl exec <copied-podname> dspmq` . You will see **acemqserver** (the Queue Manager) running.
1. To Browse a message on theNEWORDER.MQ queue that you defined earlier, execute `kubectl exec -it <copied-podname> /opt/mqm/samp/bin/amqsbcg NEWORDER.MQ`

## Review Event Streams portion

To review that the message has been correctly published to Event Streams, do the following:

>> Hugh: to be completed.


## Create API Facades

**Note** at the time of the writing of this lab, there is an issue with API Connect that must be resolved before continuing, but fortunately is easily fixed.

1. Open the main Admin console for APIC by opening a new browser tab on the Developer machine to `https://mgmt.10.0.0.5.nip.io/admin`
2. login with the credentials of `admin`/`7Ir0n-hide`

Create APIs for each of the inventory, order and AcmeMart APIs.

>**Note** the Swagger for the two ACE flows can be imported as APIs using the `From Existing Open API Service` option in API Connect.  The AcmeMart swagger can be downloaded from the main developer page and then imported, but use the `New Open API` option instead.

**For the AcmeMartUtilityAPI** you will need to modify your invoke URL to look like the following:

`http://(yourapp ip):(your port)$(request.path)$(request.search)`

Where `yourapp ip` and `your port` is the port that ICP has put your Utility App.

![](./images/cipdemo/invoke.gif)

Test your APIs in the test tool inside of API Connect.

![](./images/cipdemo/test_api.gif)

For the `AcmeMartUtilityAPI`, the quickest way to do this is open up the Assembly view of the AcmeMart and select the `/Utilities/Ping` API.  It requires no arguments.

Before:

![](./images/cipdemo/test_acmemart_api1.gif)

After:

![](./images/cipdemo/test_acmemart_api2.gif)

Testing the `Inventory` API can be done in the Assembly view by using the following value as input for `key`: `AJ1-05.jpg`

![](./images/cipdemo/store_inventory_assembly_test.gif)

The orders flow can't be (easily) tested inside of the Assembly test view as it requires a json payload.  We will test this using the flow below.

## Test the entire flow

In this part, you will explore the consumer experience for APIs that have been exposed using API Connect. Using the Developer Portal, you will log in and subscribe to the API Product and test the APIs from the portal, before testing it in a live Web Application.

This part will show the following:
+ How to subscribe to a plan in order to consume an API.
+ How to test an API from the developer portal.
+ How to consume an API from a sample test application.

1. Launch the Developer Portal from your bookmarks if you have the link saved, otherwise you  can obtain the Developer Portal URL from the API Manager. Go to your Catalog (eg. `Sandbox`), from the `Settings` menu select `Portal` to show the configuration. Copy the URL.
2. Open a new brower tab and paste the URL to launch the Developer Portal

![](./images/cipdemo/portal.png)

3. Click the `API Products` link
4. Log in with the Developer account for your portal. Remember that this account is different from the credentials you use to log in to API Management
5. Click the `API Products` link after logging in
6. Select the API product you published in the prrevious part and click `Subscribe`
7. As the result, the application is now subscribed to the product and its credentials are registered to use the Product APIs.

>In this section, you will use the Developer Portal to test the entire flow you created. This is useful for application developers to try the APIs before their application is fully developed or to simply see the expected response based on inputs they provide the API.

1. Click the logistics AcmeMartUtilityAPI 1.0.0 link on the product page
2. Check Inventory - Supply it the name of the image uploaded. Click the GET method of the API path on the left-hand navigation menu

Example Output:
```
{"Inventory":[{"color":"ultramarine color","location":"In Store","pictureFile":"AJ1-05","productDescription":"Lorem ipsum dolor sit amet, consectetur adipiscing elit. Aenean egestas nec mauris non cursus. Donec non justo lacinia, imperdiet nibh quis, laoreet ante. Sed quis luctus ligula. Donec urna libero, malesuada eu nibh vitae, facilisis pharetra quam. Donec ullamcorper porttitor bibendum. Nulla nec arcu nec metus auctor efficitur. Ut at magna condimentum, semper augue id, finibus tortor.\\n\\nLorem ipsum dolor sit amet, consectetur adipiscing elit. Aenean egestas nec mauris non cursus. Donec non justo lacinia, imperdiet nibh quis, laoreet ante. Sed quis luctus ligula. Donec urna libero, malesuada eu nibh vitae, facilisis pharetra quam. Donec ullamcorper porttitor bibendum. Nulla nec arcu nec metus auctor efficitur. Ut at magna condimentum, semper augue id, finibus tortor.\\n\\nLorem ipsum dolor sit amet, consectetur adipiscing elit. Aenean egestas nec mauris non cursus. Donec non justo lacinia, imperdiet nibh quis, laoreet ante. Sed quis luctus ligula. Donec urna libero, malesuada eu nibh vitae, facilisis pharetra quam. Donec ullamcorper porttitor bibendum. Nulla nec arcu nec metus auctor efficitur. Ut at magna condimentum, semper augue id, finibus tortor.\\n\\nLorem ipsum dolor sit amet, consectetur adipiscing elit. Aenean egestas nec mauris non cursus. Donec non justo lacinia, imperdiet nibh quis, laoreet ante. Sed quis luctus ligula. Donec urna libero, malesuada eu nibh vitae, facilisis pharetra quam. Donec ullamcorper porttitor bibendum. Nulla nec arcu nec metus auctor efficitur. Ut at magna condimentum, semper augue id, finibus tortor.\\n\\nLorem ipsum dolor sit amet, consectetur adipiscing elit. Aenean egestas nec mauris non cursus. Donec non justo lacinia, imperdiet nibh quis, laoreet ante. Sed quis luctus ligula. Donec urna libero, malesuada eu nibh vitae, facilisis pharetra quam. Donec ullamcorper porttitor bibendum. Nulla nec arcu nec metus auctor efficitur. Ut at magna condimentum, semper augue id, finibus tortor.","productID":"AJ1-05","productName":"Blue & White","qtyOnHand":"1050","rating":"2","type":"AirJordan1","typeDescription":"Air Jordan 1 (Extra Crispy)","unitPrice":"105.99"}]}
```

4. Order a product - Use the product ID selected from the check inventory call

```
curl -k -X POST \
  https://apigw.10.0.0.5.nip.io/admin-admin/sandbox/orders/v1/create \
  -H 'cache-control: no-cache' \
  -H 'content-type: application/json' \
  -H 'x-ibm-client-id: 5fa14472d2aa8f1ff5389ad20c1eed03' \
  -H 'x-ibm-client-secret: ca0986b82a21e3af67bc19ef72b7bf99' \
  -d '    {
    "accountid": "A-10000",
     "order": {
       "orderDate": "2018-11-28 17:05:39 +0000",
       "contractId": "00000100",
       "orderDetails": [
        {
           "lineItemNumber": 1,
           "productId": "AJ1-05",
           "quantity": "20"
         }
       ]
     }
    }
'
```

Example Output:
```
{"accountid":"A-10000","orderid":"6981359"}
```


5. Check events - Use the order ID from the previous call


Currently for this lab, events are being written to both Event Streams on your ICP and in the Cloud.  We're still working through some of the challenges with the ES on-premesis but we can still show events for the demo that are running on the IBM Cloud.  Execute this command to get a feel for what those look like.

```
curl -X GET \
  'https://am-utilityapi.eu-gb.mybluemix.net/api/Events/findOne?filter=%7B%22where%22%3A%20%7B%22purchaseOrder%22%3A%20%225688471%22%7D%7D' \
  -H 'cache-control: no-cache' \
  -H 'x-ibm-client-id: af0a85e4-92fe-4b87-8545-f536fbf811a3' \
  -H 'x-ibm-client-secret: M6hQ5nR0kT2jO6mU5gA3dS3yN7uT8vY7eG3jH4xN1pS7vM7hN3'
```
Example Output:
```
{"id":"f58d732c494dc8ce9c1d2dbdba8817dd","purchaseOrder":"5688471","shipTo":{"name":"Test Account","street":"1060 West Addison","city":"Chicago","state":"IL","zip":"60680"},"billTo":{"name":"Test Account","street":"1060 West Addison","city":"Chicago","state":"IL","zip":"60680"},"item":{"partNum":"AJ1-03","productName":"Red, Black & Green","quantity":"1","price":"103.99","shipDate":"2019-01-08T19:46:28.314Z"},"status_code":500,"last_update":"2019-01-08T19:56:01.479Z","history":[{"type":"initial","timestamp":"2019-01-08T19:46:29.350Z","topic":"kafka-nodejs-console-sample-topic","partition":0,"offset":334,"key":null},{"type":"update","timestamp":"2019-01-08T19:48:03.328Z","topic":"acmemart_update_order","partition":0,"offset":151,"key":[75,101,121]},{"type":"update","timestamp":"2019-01-08T19:50:01.362Z","topic":"acmemart_update_order","partition":0,"offset":153,"key":[75,101,121]},{"type":"update","timestamp":"2019-01-08T19:52:01.403Z","topic":"acmemart_update_order","partition":0,"offset":155,"key":[75,101,121]},{"type":"update","timestamp":"2019-01-08T19:54:01.438Z","topic":"acmemart_update_order","partition":0,"offset":157,"key":[75,101,121]},{"type":"update","timestamp":"2019-01-08T19:56:01.479Z","topic":"acmemart_update_order","partition":0,"offset":159,"key":[75,101,121]}]}
```




## Analyze API consumption

### Launch the API Manager and navigate to Analytics

1.  Sign on to the API Manager UI of IBM API Connect

2.  Click on the tile entitled `Manage Catalogs`.

3.  Click on the tile representing the catalog you have published the REST facades to (eg. `Sandbox`)

4.  From the catalog configuration screen, hover over the icons on the left, and click on the `Analytics` tab.

![](./images/cipdemo/analytics_tab.gif)

### Explore the Analytics

1. After clicking the `Analytics` tab, you are presented with a list of dashboards.

    Those tagged with `Admin` are pre-defined by IBM. You can clone (copy) a dashboard and then edit it - instructions for doing this are provided later below. You can add your own dashboards from scratch, and you can import dashboards that have been exported from another API Manager.

2. Click on the `API Default` dashboard.

3. This default dashboard gives some general information about Errors, Status Codes, Response Times and number of API Calls.

    Take a while to explore this dashboard and its visualisations. Note, and experiment with, the following:
    1. Use your browser's zoom capability to zoom in or out, until the screen size is comfortable for you.
    2. At the top right of the screen, click the time range (the default is `Last 15 minutes`) to examine the Time Range options. (Notice that when you are choosing the Time Range, you can also set the autorefresh time by clicking `Auto-refresh`.)
    3. Note that when you click on an area of some of the visualisations, this adds a filter to the list of filters along the top of the screen. Try this with the `Status Codes (Detailed)` graph. To remove a filter, hover over it and click the dustbin icon. Once you have one or more filters defined, note that you can toggle the `Actions` option on the right-hand side, to enable and disable filters.
    4. Note the Search box at the top, which - as it says - uses "lucerne query syntax". Try searching for _status_code:200_, and _status_code:300_. (Make sure you hit `Enter` after changing the search term.)
    5. Take time to hover over various areas of the visualisations, and click through, to investigate other options.


4. When you have finished exploring the `API Default` dashboard, click `Clone` at the top to make a copy of this dashboard. Give the cloned dashboard your own unique name (eg "My Clone of API Default") and confirm it.
5. Notice now that the dashboard name along the top has changed, and you also have an `Edit` button along the top. Click this `Edit` button.
6. You are now able to edit the look and feel of the dashboard as a whole - these changes should then be saved for you under its new name.

    Explore the following options:
    1. Use the `Add` button to add a visualisation or a Saved Search.
    2.  Use the four-arrows icon to move a visualisation.
    3.  Use the cross icon to remove a visualisation.
    4.  Drag the bottom-right corner of a visualisation to resize it.
    5.  Explore other options.

7. Once you have finished editing your dashboard, click `Save` to save it, and confirm.
8.  Click `Dashboard` in the dashboard breadcrumb trail at the top, to go back to the list of dashboards.


9.  Notice that your dashboard is now in the list. It has been tagged with `Catalog`, to distinguish it from the admin-provided ones.

10.  Explore the other dashboards to see what visualisations they include.


### Conclusion
You have put together the main building blocks for the combined CIP Demonstration.  The only things left are to plug in the mobile app to make these api calls, and make a minor change to the microservices to use the on-premises based event streams versus the cloud.  There is also a Watson Assistant (aka Conversations) bit that will be added as well. Keep an eye out for further sessions, as all of these items will be covered in a forthcoming remote enablement session later in this quarter.


**End of Lab**
