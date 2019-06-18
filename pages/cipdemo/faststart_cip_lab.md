---
title: FastStart 2019 Lab - CIP
toc: false
sidebar: labs_sidebar
folder: pots/cipdemo
permalink: /cip-demo-lab.html
summary: Deeper dive into the Cloud Integration Platform
applies_to: [developer,administrator,consumer]
---

# Explore the Cloud Integration Platform
===============

## Overview

The goal of this exercise is to provide you some hands on with the platform and build your skills with the platform with the superior goal of creating a demo environment everyone is able to build and use on their own.

This lab guide will lay out the scenario for you, along with the requirements of what you are to complete.  This guide is a bit different than other labs, such that it will (for the most part) not be a step by step walkthrough of what to complete, rather will give you a list of requirements to complete, and it is up to you and your team to deliver on those.

This lab will be a team effort.  It is highly suggested that you form your teams of folks who can cover the entire I&D Portfolio - especially: App Connect Enterprise (using Toolkit), API Connect, Messaging (MQ and ES).  Its helpful also to have someone who knows `kubectl` and `docker` commands and has some background with ICP or Kubernetes.


Lab Overview
-------------------------------------------

You will be building a demo environment that supports the "AcmeMart" demo scenario.  There are several components that make up this demo, but the key areas in scope for this lab are as follows:

*  Two App Connect Enterprise flows that interact with IBM Cloud based APIs
*  AcmeMart Node.js based APIs
*  On-premises based MQ and ES based assets
*  API Facades for RESTful assets to be created in API Connect

All assets created are to be implemented on the given CIP environment provided for you.  All pre-requisites and utilities should be on the environments provided.  If there are other utilities and tools you would like to use to support you during this exercise, you may do so at your own risk.


Lab Environment Overview
-------------------------------------------
You have been provided a pre-installed environment of the Cloud Integration Platform with the base charts for CIP already deployed and configured.  This includes specifically:

* ICP Main Portal running on `https://mycluster.icp:8443`
* CIP Platform Navigator running on: `https://mycluster.icp/integration`

You should be able to access all portals from the Platform Navigator, but if you need to access the URL directly, they are provided below:

* App Connect Enterprise Manager Portal running on `https://mycluster.icp/integration/instance/ace1`
* API Connect API Manager Portal on `https://mgmt.mycluster.icp.nip.io/manager`
* MQ Portal on `https://mycluster.icp:31681/ibmmq/console/`
* Event Streams on `https://mycluster.icp/integration/instance/es`

2. The environment you are using consists of 6 different nodes.  5 of which are ICP Nodes and one is a developer image that you will be using to access the ICP User Interfaces.  You can access this VM directly using the Skytap interface to use the X-Windows based components. All of the base CIP Charts are already installed and configured.

3. **VERY IMPORTANT** It is very important you do not suspend your lab environment.  We have seen cases when the environment goes into suspend mode, the Rook-Ceph shared storage gets corrupted.  Also it is a good practice that you shut down ICP before powering down your enviroment.  A script has been included to handle that for you that will be explained in the coming sections.  When you power down your environment, you can safely use the `power off` option as the shared storage can interfere with the normal graceful shutdown method.  So far using power off hasn't caused any problems that we are aware of.  If you find that your API Connect environments are not coming up properly, it suggested that you execute the `./icpStopStart.sh stop` script (you may need to execute it more than once).  When it stops, go ahead and execute the `./icpStopStart.sh start`.  As indicated previously, this may take ~30 minutes or so to come up.

3. You are also able to SSH into your environment.  You can find out the exact address to SSH to in your Skytap Environment window under `networking` and `published services`.  The Published Services set up for you will be for the SSH Port (22) under the `CIP Master` node.  The published service will look something like `services-uscentral.skytap.com:10000`.  Your environment will have a different port on it.  You can then SSH to to the machine from your local machine.

4. Additionally, you can access the machine direct via the Skytap UI.  This is functional, but can be cumbersome to work with as its not easy to copy and paste into and out of Skytap, especially for the non X-Windows based environments.

5. Password-less SSH has also been enabled between the Master node and the other nodes in the environment.  **note** a table with the environment configuration is provided below.  Credentials for each machine are `root`/`Passw0rd!`.

| VM        | Hostname  | IP Address | # of CPU | RAM    | Disk Space (LOCAL) | Additional Notes |
|-----------|-----------|------------|----------|--------|--------------------|------------------|
| Master    | master    | 10.0.0.1   | 24       | 39 GB  | 425 GB             | hostname mycluster.icp|
| Worker 2  | worker-2  | 10.0.0.3   | 12        | 47 GB  | 600 GB             |       |
| Worker 3  | worker-3  | 10.0.0.4   | 12        | 47 GB  | 600 GB            		  | 		           |
| Worker 4  | worker-4  | 10.0.0.5   | 12        | 47 GB  | 600 GB             		  | 		            |
| Worker 5  | worker-5  | 10.0.0.6   | 12        | 47 GB  | 500 GB                            |                  |
| Developer | developer | 10.0.0.9   |          |        |                                  |                  |


6. If it is not done already, power up your Environment.  It could take about 5 minutes for all nodes to start up.  The master node takes the longest to come up, so if you can see the login prompt from the Skytap UI on the master node, then you are good to go.

7. SSH into the Master Node or use the Skytap UI.  In the home directory of root (`/root`) there is a script called `icpStopStart.sh`.  Run this script by executing `./icpStopStart.sh start` Note that it takes around 30 minutes for the ICP Services to come up completely.

8. You can tell if the start of ICP is complete by checking using one of these two methods:
	- Click on the Developer Machine, and it will take you directly to the Developer Machine running X-Windows.  Should you need to Authenticate, you can use the credentials of `student`/`Passw0rd!`. Bring up the Firefox browser.  Navigate to the main ICP UI by going to `https://mycluster.icp:8443`.  The credentials again are `admin/admin`.  If you can log in and see the main ICP Dashboard, you are good to go.
	- Alternatively, you can SSH to the Master node.  Execute a `cloudctl login` from the command line.  If all there services are up, it will prompt you for credentials (use `admin`/`admin`) and setup your kubernetes environment.
8. The best place to do your Kubernetes CLI work is from the Master node.  Again, Before you can execute any of the `kubectl` commands you will need to execute a `cloudctl login`.
9. Next step is to find The Platform Navigator UI can be use to create and manage instances of all of the components that make up the Cloud Integration Platform.
10. You can access the Platform Navigator using the browser on the developer machine.  The URL for the navigator was set up in this environment as: `https://mycluster.icp/integration`.  You might be asked to authenticate into ICP again, but once you do that you should now see the Platform Navigator page.
11. The Platform Navigator is designed for you to easily keep track of your integration toolset.  Here you can see all of the various APIC, Event Streams, MQ and ACE instances you have running.  You can also add new instances using the Platform Navigator.


### Key Concepts - Troubleshooting / Recovery

There are times where things may not be going right, so your best bet is to use `kubectl` commands to uncover more information about what is going on. A list of useful commands are provided below:

 - `kubectl get pods -n <some-namespace>` This command shows all of the pods in a given name space.  Here you will see if any pods are up, down, errored or otherwise in transition.
 - `kubectl describe pods <some-pod> -n <some-namespace>` This will provide verbose information about a given pod.  You can use the `describe` command for other objects.
 - `kubectl logs <some-object> -n <some-namespace>` This command will work with other objects as well.  You can also tail the logs by using the `-f` switch at the end.

 Logging is also done inside of ICP using the ELK stack.  You can access the logging inside of ICP and use Elastic Search commands to drill into things.  For example, if you were to bring up Kibana and enter in a search like `kubernetes.namespace:<your namespace>`.  Using this along with the `kubectl` commands gives you a lot of power to dig into root cause.


Lab Requirements
-------------------------------------------

## ACE Integration Assets

1. The ACE integration assets have already been populated into this lab for you. (You can also find them in this repository: `https://github.com/ibm-cloudintegration/techguides/pages/cipdemo`.)
2. Below is a description of each of the files that are relevant to the ACE portion of this lab.

| File                           | Description                                                                                      |
|--------------------------------|--------------------------------------------------------------------------------------------------|
| ACEflows.zip             | ACE Project Interchange export of integration flows - allowing you to restore the code for the supplied ACE flows and APIs, in case anything goes wrong ! |
| order.json             | a sample order file |
| generateSecret.sh     | a script file that generates an ICP Secret, for use when deploying BAR files (Helm Charts) into ICP|
| serverconf.yaml     | skeleton file, used by generateSecret.sh|
| setdbparms.txt     | parameter file, used by generateSecret.sh, which you will edit|
| truststorePassword.txt     | parameter file, used by generateSecret.sh|



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

5. The AcmeMart Microservice is already pre-configured, but we highly recommend to run the configuration yourself, if possible. Execute this command to delete the existing configuration: `kubectl delete namespace acmemartapi`
6. Execute this command:  `docker build . -t acmemartutilityapi`.  The image and its dependencies will be downloaded.
7. In the ICP UI, starting from the top left hamburger icon select `Manage` -> `Namespaces`.  Create a new namespace and call it `acmemartapi`.  Using the `ibm-anyuid-hostpath-psp` security policy is fine for this one.
8. Tag your new image by executing this command:  `docker tag acmemartutilityapi mycluster.icp:8500/acmemartapi/acmemartutilityapi:v1.0.0`
9. Push the docker image out to your ICP instance by issuing this command `docker push mycluster.icp:8500/acmemartapi/acmemartutilityapi:v1.0.0`.
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


## Prepare for Event Streams

In this section you will prepare for the integration of Event Streams and ACE.

### Define and capture Event Streams Configuration Information

Start by working with Event Streams, to create a topic and to define and capture connection information:
1. Navigate to the Event Streams dashboard:
 - Open a browser session to the Cloud Integration Platform home: `https://mycluster.icp/integration/`
 - Under `Messaging`, select the `es` link, to open the Event Streams Dashboard.
 - (If you see an error screen similar to that shown below, then simply select the `Open es` link in the middle.)
		![](./images/cipdemo/open_es_error.jpeg)
 - Provide or accept the credentials (Username **admin** Password **admin**) , then `Log In`.

1. Now you are in the main Event Streams dashboard. Select the `Topics` tab.
1. Select `Create Topic`, to start creating a new topic:
 - You will be taken through 4 screens where you specify basic Topic configuration. Note the` Advanced` option : this would present all the parameters on one long screen. You can explore the Advanced options if you like, but for this lab we assume you will just specify the basic configuration parameters.
  - For the `Topic name`, we recommend **NewOrder**. This is the topic to which, in this lab, a message will be published by ACE, each time a new order is created. If any application wants to subscribe to this message then they must be told what this value is.
 - Leave the `Partitions` as **1**. (For scalability in Production, you might specify **3** or more.)
 - You can specify whatever `Message Retention` you like; we recommend leaving it as **A week**.
 - The `Replicas` setting is used for High Availability; we recommend you leave it to default.
  - Select `Create Topic` at the end. You will be taken back to the Topics screen.

Now you will create and extract Event Streams connection information, for use later on.
4. Select `Connect to this cluster`, to pull in the window that allows you to create and copy configuration information.

1. Make a note of the value of the `bootstrap server`. (Store it in a temporary file, or just write it down.) Make sure you note it correctly, because you must specify this exactly in the ACE configuration later.
1. Use the correct button to download the `PEM Certificate`. Store it in `/home/student/Downloads`.
		![](./images/cipdemo/bootstrap-server-and-certificate.png)
1. Use the section on the right to create an API key:
 - First `Name your application`. This is used, within Event Streams, to report connections and activity, but it is not used further in this lab session. You could specify **ACE-Connection**, or **ordersFlow**, for example.
 - Specify `Produce, consume and create topics`. Although in this lab session we ask you only to produce messages, specifying this allows you to extend the lab and do more later, if you want. In a Production system you could use this to restrict access.
 - Select the slider, to specify `All topics`. Although in this lab session we ask you to use only the topic you created above,  specifying `All topics` allows you to extend the lab and do more later, if you want. In a Production system you could use this to restrict access.
 - Ensure that the slider specifying `All` consumer groups is enabled. In a Production system you could use this to restrict access.
 - After selecting `Generate API key`, make sure you download the API key (a JSON file called **es-api-key.json**) to _/home/student/Downloads_.
 - Don't forget to select `Create a new API key`, to make it all happen.

&nbsp;

### Generate a Secret object

Now you will generate a "Secret" object. This is a Kubernetes construct that lets you store and manage sensitive information, such as passwords and certificates. In this lab, a Secret is used to store connectivity information for connecting securely from an ACE Helm Release (aka Integration Server) to your instance of Event Streams.

(More information on Secrets can be seen here:
https://kubernetes.io/docs/concepts/configuration/secret/ )

1. Prepare the PEM file you downloaded earlier.
 - On the Developer Machine, open a "Files" session.
 - Move the PEM certificate file that you downloaded from Event Streams earlier (probably called **es-cert.pem**) from _/home/student/Downloads_ to _/home/student/generateScript_.
 - Rename this PEM certificate file in _/home/student/generateScript_ to the following: **truststore-escert.crt** (yes, this means that it will no longer be a PEM file). Note: that this name structure must be of the form **truststore-ALIASNAME.crt**, where ALIASNAME will be generated as the alias for the certificate. Note down your specific  ALIASNAME (**escert**), because you will need to specify this later when deploying a BAR file.
1. Go back to _/home/student/Downloads_, open the downloaded JSON file **es-api-key.json** in your favourite editor and copy the API key to the clipboard. Note: copy only the API Key, not the quotes around it.
![](./images/cipdemo/ace-copy-api-key.jpg)

1. On the Developer Machine, open a Terminal session.  Note that you will be signed in as _student_, and be in directory _/home/student/_.
1. Change to working directory  _/home/student/generateScript_. This directory contains a tool called  _generateSecret.sh_, which will generate the Secret you need. It also contains some files that form input into that tool. This directory now also has your renamed PEM file (**truststore-ALIASNAME.crt**) in it, which also forms input into the tool.
1. Use either `gedit setdbparms.txt` or `vi setdbparms.txt` to edit the **_setdbparms.txt_** file.
    - Replace the characters `<over-write with API Key>` with the API key that you copied to the clipboard a moment ago. Note that this must be done accurately, otherwise the connection from ACE to Event Streams will not work. The content of the file should look like this:
 ```
 kafka::KAFKA token TYu9y5iTqBxKeCDuko-BBCnaMpDn2zIZYJp7uYUzGFWh
 IntSvr::truststorePass thisispwdfortruststore password
 ```
    - The first line means "when ACE uses its Kafka client to connect, use this API key as the password". (**token** is the userID for that password, but it is not used.)
    - The second line means "when ACE refers to the identity _IntSvr::truststorePass_, it will use the password **password**. (**thisispwdfortruststore** is the userID for that password, but it is not used.) Note that in the set of configuration files, **truststorePassword.txt** has already been defined, containing the matching value **password**.
    - `Save` your changes and exit the editor.
1. Sign into the CIP namespace and run the tool to generate the Secret.
 - In the Terminal session, execute `sudo cloudctl login`.
 - Provide the password for student: **Passw0rd!**.
 - Ensure that the API Endpoint **https://mycluster.icp:8443** is specified. If a different one is specified, execute `sudo cloudctl logout` and try again.
 - Provide the CIP userid: **admin** with  password **admin**
 - Set the namespace context to **ace**. (This is because the Secret we are about to generate must be in namespace **ace**.)
 - Execute `./generateSecret.sh orders-secret`. This script takes in the various configuration files (including the PEM certificate and the API key) and then uses _kubectl_ to generate the Secret with name **orders-secret**. You should see a success message `secret/orders-secret created`.
1. Check that the secret has been created, using the UI.
 - In a browser session, navigate to the ICP Portal: https://mycluster.icp:8443.
 - Using the hamburger menu, navigate to `Configuration` -> `Secrets`.
 - Confirm that Secret **orders-secret** has been created, and click it to open and check that it is in namespace **ace**.

You have now finished preparing Event Streams, and you have created the artefacts needed by ACE ready to connect to Event Streams.

## Configuring remote MQ to permit ACE connection

The `orders` API provided by ACE will put one message onto a queue on a local Queue Manager (called **acemqserver**), and also the same message onto a queue on a remote Queue Manager (called **mq**). This lab session uses those two Queue Managers, to   illustrate the configuration differences between a remote and a local Queue Manager.

You do not need to perform any extra configuration on the local Queue Manager **acemqserver**. The `orders` API will be able to connect to it.

In this section, you will perform the necessary configuration on the remote Queue Manager **mq**, to permit ACE to connect to it.

### Work with MQ artefacts

The remote Queue Manager called **mq** has already been created and deployed, in its own Helm Chart. You will now use the MQ Console to perform some configuration.
1. Open the MQ Console for the Queue Manager in one of these  ways:
  - Point a browser session at the ICP Main Portal: `https://mycluster.icp:8443`, open `Workloads` -> `Helm releases`, find the Helm Release called `mq` and on the right `Launch` -> `console-https`.
  - Point a browser session at the ICP4I Platform Navigator: `https://mycluster.icp/integration`, and under `Messaging` select `mq`.
  - Point a browser session directly at the MQ Console: `https://mycluster.icp/integration/instance/mq`
3. Click the Queue Manager name `mq` to highlight it. (This is the name of the Queue Manager already created in this pod). Select `Properties`.

  ![](./images/cipdemo/ace-mq-properties.jpg)

5. On the `Communication` tab, find the `CHLAUTH Records` property and make it `Disabled`.

  ![](./images/cipdemo/ace-chlauth-disabled.jpg)

1. Don't forget to `Save` and then `Close`.

You will now add a new queue, a new channel, and an authentication record.

5. At the top right, click `Add Widget`, then select your Queue Manager `mq` and select the `Queues` widget.
  -  In your new Queues widget, click on `Create (+)` to create a new queue called **NEWORDER.MQ**, of type  `local`.
  - You have just created a queue, onto which messages can be put.
1. At the top right, click `Add Widget` again, then select your Queue Manager `mq` and select the `Channels` widget.
  -  In your new Channels widget, click on `Create (+)` to create a new channel called **ACE.TO.mq**, of type `Server-connection`.
  - Having created it, click `ACE.TO.mq` to select it, and then click `Properties`.
  - On the MCA tab, for `MCA User ID` specify **mqm**. This forces this userID to be used when a connection is attempted. For this lab session it makes for an easy connection; in a Production environment more strict security should be applied.
	![](./images/cipdemo/ace-specify-MCA.jpg)
  - You have just created a channel, which will be used by the MQ Client built into ACE Integration Server when its flows want to connect to this Queue Manager.
1. At the top right, click `Add Widget` again, then select your Queue Manager `mq` and select the `Channel Authentication Records` widget.
  -  In your new Channel Authentication Records widget, click on `Create (+)`.
  -  Specify `Rule Type` = **Block**, and `Identity` = **Final assigned user ID** - click `Next`.
  - Specify `Channel profile` = **ACE.TO.mq** (case-sensitive) and `User list` = **nobody** - click `Next`.
  - Leave the `Description` and `Custom` fields blank - click `Create`.
  - What you have just created will block user **nobody** from this channel, thus allowing all other users to use this channel. For this lab session it makes for an easy connection; in a Production environment more strict security should be applied.
1. At the top right, click `Add Widget` again, then select your Queue Manager `mq` and select the `Authentication Information` widget.
  -  In your new  Authentication Information widget, click on the cogwheel to configure the widget, and select `System objects` -> `Show`.
	![](./images/cipdemo/ace-authinfo-cogwheel.jpg)
  - Click the system-provided Authinfo called `SYSTEM.DEFAULT.AUTHINFO.IDPWOS` and then click `Properties`.
  - On the `User ID + password` tab, for both `Local connections` and for  `Client connections` specify **None**.
	![](./images/cipdemo/ace-idpwos.jpg)
  - Don't forget to `Save` and then `Close`.
  - Those **None** settings mean that the authentication of the client is not checked. For this lab session it makes for an easy connection; in a Production environment more strict security should be applied.
1. Back on the Local Queue Managers widget, make sure the Queue Manager `mq` is selected, then click the ellipsis, click `Refresh Security...` and then select `Connection authentication`. This will make sure that the security changes you have just configured will now take effect.

  ![](./images/cipdemo/ace-refresh-security.jpg)

1. Your MQ Console should show your new widgets and your new artefacts, thus:
  ![](./images/cipdemo/ace-mq-console-details.jpg)

Finally, you will check which port the MQ Listener is listening on, thus:

11. At the top right, click `Add Widget` again, then select your Queue Manager `mq` and select the `Listeners` widget.
  -  In your new Listeners widget, click on the cogwheel to configure the widget, and select `System objects` -> `Show`.
	![](./images/cipdemo/ace-authinfo-cogwheel.jpg)
  - You should see the system-provided Listener called `SYSTEM.LISTENER.TCP.1`. Make a mental note of the port for this listener (almost certainly it will be the MQ default of **1414**).
	  ![](./images/cipdemo/ace-showing-listener.jpg)
1. In a browser session, navigate to the ICP Portal: https://mycluster.icp:8443.
  - Using the hamburger menu, navigate to `Configuration` -> `Helm releases`.
  - Click the `mq` release to open it.
  - Towards the bottom, you should see the Service called `mq-ibm-mq`, and under `PORT(S)` you should see the port mentally noted earlier (probably **1414**).
  - Write down the associated mapped port number, which will be **3xxxx**, as shown below:

  ![](./images/cipdemo/ace-determine-mq-listener-port.jpg)

You have now identified that the queue manager `mq` is listening on port **1414** within its pod, and that this is mapped to the external port **3xxxx**. The latter number (**3xxxx**) is the one that you will need to specify when connecting to this Queue Manager.

You have now finished preparing the remote Queue Manager `mq`, to allow the MQ Client within ACE to connect to it.


## Modify the `orders` API within ACE

The REST APIs within ACE, that form part of the overall solution, are mostly already written for you. You will now make changes to one of those REST APIs (namely the `orders` API) to make it put messages on MQ queues and publish a message to an Event Streams topic).

1. On the Developer Machine, go back to the Terminal session and navigate to directory _/home/student_.
1. Start the Ace Toolkit by executing `sudo ./ace-v11.0.0.3/ace toolkit`.
1. Make sure that you specify the correct workspace _/home/student/IBM/workspace_ as shown:

 ![](./images/cipdemo/ace-select-workspace.jpg)

Note that the ACE Toolkit may start as a tiny window on the screen. Use the cursor to grab the corner of this screen and expand it.

 ![](./images/cipdemo/ace-tiny-toolkit-start.jpg)

4. You should see, in the Application Development window on the lefthand side, three REST APIs and their artefacts:
 - **Customer**, which this lab does not use
 - **orders**, which you are about to change and then deploy
 - **storeinventory**, which you will not change and is already deployed (as **inventory**)
	 ![](./images/cipdemo/appl_dev_window.png)

### Modify the `orders` subflow

The `orders` subflow forms the body of the work that ACE will do when a new order is created and the orders API is called.

You will now modify this subflow, by adding four operations (nodes) following the App Connect REST Request operation (node).
1. The first node will strip the HTTP Headers. This is because before you do any further work with this message in this subflow, you must remove this header.
1. The second new node will put a message to an MQ queue on a local Queue Manager (inside the same pod as the ACE Integration Server).
1. The third new node will put a message to an MQ queue on a remote Queue Manager (inside a separate pod).
1. The fourth new node will publish a message to a topic in EventStreams.

Note: this lab session uses both a local Queue Manager and a remote Queue Manager, to illustrate the differences between the two. In a real Production environment, good design practice recommends always using a remote Queue Manager, to provide maximum componentisation of the infrastructure.

Here is what the **orders** subflow will eventually look like. Detailed instructions follow.

 ![](./images/cipdemo/ace-orders-subflow-canvas.jpg)

1. Drill down `orders` -> `Resources` -> `Subflows` and open `New_Order.subflow`.
1. From the ACE palette, drag and drop an `HTTPHeader` node, two `MQOutput` nodes and a `KafkaProducer` node. Position them and connect them as shown in the screenshot above.
1. Select the `Http Header` node. Configure its properties thus:
 - On the `HTTPResponse` tab, select **Delete HTTP Header**.

 ![](./images/cipdemo/ace-delete-httpheader.jpg)

1. Select the first `MQOutput` node. Configure its properties thus:
  - For `Queue Name` specify **NEWORDER.MQ**. This is the queue onto which the message will be put. Note: in this instance we are hard-coding this queue name; typically it will be parameterised (for ACE specialists: this parameterisation uses _LocalEnvironment.Destination.MQ.DestinationData.queueName_).
  - For `Connection` select `Local queue manager`, because this is connection to the local Queue Manager (running inside the same pod that is also running ACE).
  - For `Destination queue manager` name specify **acemqserver** (case sensitive). This is the name of the local Queue Manager, which will be defined when this flow and its Integration Server and Queue manager are deployed . Note: in this lab we are hard-coding this Queue Manager name; typically it will be parameterised (using an MQ Policy).

 ![](./images/cipdemo/ace_mqoutput.jpg)

2. Select the second `MQOutput1` node. Configure its properties thus:
	 - For `Queue Name` specify **NEWORDER.MQ**. This is the queue onto which the message will be put. Note: in this instance we are hard-coding this queue name; typically it will be parameterised (for ACE specialists: this parameterisation uses _LocalEnvironment.Destination.MQ.DestinationData.queueName_).
	 - In the `MQ Connection` tab, specify the following MQ Client connection properties. This is a connection to a remote Queue Manager, running on a separate pod in the cluster. Note: in this lab we are hard-coding these connection details; typically it will be parameterised (using an MQ Policy).
      - `Connection`: **MQ client connection properties**
      - `Destination queue manager name`: **mq** (case-sensitive) - this is the name of the remote Queue Manager
      - `Queue manager host name`: **10.0.0.1** - this is the access IP address for the relevant pod
      - `Listener port number`: **3xxxx** - this must match the mapped port number that you determined earlier by looking at the Helm Release
      - `Channel name`: **ACE.TO.mq** (case-sensitive)- this must match the channel name that you created earlier by using the MQ Console

	 ![](./images/cipdemo/ace_mqoutput1.jpg)

1. Select the `KafkaProducer` node. Configure its properties thus:
 - Enter the `Topic name`, matching what you defined within the Event Streams configuration earlier - we recommended **NewOrder**. Note: in this instance you are hard-coding this topic name; typically it will be parameterised (for ACE specialists: the parameterisation uses _LocalEnvironment.Destination.Kafka.Output.topicName_).

 - For the `Bootstrap servers`, specify the value you noted in the Event Streams section earlier. It looks like **10.0.0.1:30633**.
 - For Acks specify **All**. This defines the number of acknowledgements to request from the Event Streams server before the publication request is sent. **0** is equivalent to similar to 'fire and forget'; **1** waits for a single acknowledgement; **All** waits for acknowledgements from all replicas of the topic (providing the strongest available guarantee that the message was received).
 - Change the `Timeout` to **5** secs (so that if it fails, you will only have to wait 5 seconds before you see the failure)
 - For `Security Protocol` specify **SASL_SSL**. This is because Event Streams requires this.
 - For `SSL protocol` specify **TLSv1.2**. This is because Event Streams requires this.

   ![](./images/cipdemo/ace_kafka_producer.png)

8. `Save` your flow.
9. Create a BAR (Broker Archive) file in a project called **Barfiles**, thus:
  - Right-click in the Application Development window and select `New` -> `Bar file`.
  - Call it **orders**.
  - In the `Prepare` tab, ensure that the REST API called **orders**  is selected.
  - Select `Build and Save`.

The BAR file containing the changed `orders` API is now ready for deployment. It is called **orders.bar**, and resides in the ACE workspace (which is _/home/student/IBM/workspace/BARfiles_).


## Deploy the `orders` BAR File

In this section you will deploy the `orders` BAR file (the BAR File you changed earlier) to the CIP environment.

Deployment of a BAR file includes the creation of the Integration Server in which it will run, and you do this by configuring and deploying a Helm Chart. This Helm Chart includes all the details of the Integration Server, as well as the BAR file itself.

In a DevOps environment, you would expect to configure and deploy the Helm Chart programmatically. In a Development environment, such as this lab, the typical way to deploy a BAR File (and create the Integration Server) is to start from the ACE Dashboard.

>_**Note:**_ When deploying, you will create a unique hostname for each BAR file; this is referred to as the `NodePort IP` setting when configuring the Helm Release. For example: **orders.10.0.0.1.nip.io** and **inventory.10.0.0.1.nip.io**. The **10.0.0.1.nip.io** portion specifies that an on-demand service (**nip.io**) is used to route to **10.0.0.1** (the IP address of the ICP proxy node). The whole value of `NodePort` must be unique; it points to the HTTP listener for the specific Integration Server.

1. Use one of the following methods to get to the ACE Dashboard.
 - Either go directly by opening a browser session to **https://mycluster.icp/ace-ace1**
 - Or start with the Platform Navigator: **https://mycluster.icp/integration/**, and select `ace1`. Note: if this appears to fail as shown in the following screenshot, simply select `Open ace1` on the failure screen and it should work.

	  ![](./images/cipdemo/open_ace_error.jpeg)


2. On the ACE Dashboard, make sure you are on the Servers tab.

3. Follow these steps carefully, to deploy the `orders` flow that you have just changed.
 - From the ACE Dashboard, select `Create` to start the process.

 - Browse to _/home/student/IBM/workspace/BARfiles_, select the BAR file `orders.bar`, and `Ccontinue`.
 - You will be presented by the Add Server window, showing the `Content URL`, and the `namespace` **ace**.  The `Content URL` defines the location, in ICP terms, of where the BAR file is.

	  ![](./images/cipdemo/ace-add-server.jpg)

    - Use the button to copy the contents of the `Content URL` to the clipboard, because you will need it shortly.
    - The namespace **ace** is being proposed by ACE; you should make a mental note of it.
    - Select `Configure Release` to continue.
 - ACE now selects the correct Helm Chart from the Catalog, and opens the ICP configuration pages. You can scroll through the information if you want. At the bottom of the window, select `Configure` to continue.
 - For the `Helm Chart name`, we recommend **orders**. This name will be used in many of the ICP artefacts, so a meaningful name is good. This name will also be used as the default for some of the later properties (for example the name of the Integration Server).
 - For the `namespace`, select **ace**, because that was proposed by ACE earlier.
 - Ignore the `NodePort` under `Quick start` and select `Advanced` instead. (You will complete the `NodePort` shortly.)
 - Into the `Content Server URL` field, paste the contents of the `Content URL` from the clipboard. This vital piece links the BAR file to this Helm Chart.
 - Tick the `Local default Queue Manager` option. This specifies that this deployment will include a local queue manager.
 - For the `Secret name` specify **orders-secret** as prepared earlier. This Secret adds sensitive configuration information (such as API key and a certificate) to this Integration Server, so that it can communicate with Event Streams.
 - For the `Image pull secret` specify **sa-ace**. This Secret was created when the Cloud Pack for Integration was installed, and it contains the credentials for ICP to access its private docker repository.
 - Untick `Enable persistence`. This is because provisioning volumes for persistence in this particular lab environment is sometimes troublesome. For a lab environment, persistence is not required for the MQ Queue Manager. In a Production environment, you would probably enable persistence.
 - For the `NodePort`, we recommend **orders.10.0.0.1.nip.io**. We recommend that the first part of the name (**orders** in this case) is identical to the `Integration Server name`, because there is a 1 to 1 relationship here.
 - For the `Queue manager name`, specify **acemqserver**. This defines the name that ICP will give to the associated local queue manager (which matches what we specified in the `MQOutput` node earlier).
 - For the `Certificate alias name`, specify **escert**. This is used by the Integration Server when it connects to Event Streams. You defined the value **escert** earlier, when you created the Secret.
 - Note that you could also specify the `Integration Server name`. However, we recommend that you leave this blank, so that the Helm Chart name (**orders** in this case) is used.

 ![](./images/cipdemo/ace_hc_5.png)
 ![](./images/cipdemo/ace_hc_6b.png)
 ![](./images/cipdemo/ace_hc_8.png)
 ![](./images/cipdemo/ace-disable-persistence.jpg)
 ![](./images/cipdemo/ace_hc_9.png)
 ![](./images/cipdemo/ace_hc_10.png)

  - Leave the remaining settings as defaults and then click `Install` at the bottom.
  - Your Helm Chart will now install.

> **Tip:** when you see the on the "Installation started" screen, click on the little cross top right to cancel (see screenshot below). This will mean that your Helm Chart, with all the configuration you have just typed in, remains in the browser. In turn, this means that, if something goes wrong during installation, you can a) easily check what parameters you typed and b) easily try the installation again.

 ![](./images/cipdemo/ace-installation-started.jpg)

4. Return to the ACE Dashboard and confirm that the `orders` Integration Server has been correctly deployed. You should wait a few seconds to a handful of minutes, for the deployment to succeed fully. Use the `Refresh` button.

### Some help for ACE, Event Streams and MQ Problem Determination
If the deployment does not seem to succeed, use kubectl to perform problem determination, as follows:

1. Sign into the relevant ICP namespace:
 - In the Terminal session, execute `sudo cloudctl login`.
 - Provide the password for student: **Passw0rd!**
 - Ensure that the API Endpoint **https://mycluster.icp:8443** is specified. If a different one is specified, execute `sudo cloudctl logout` and try again.
 - Provide the CIP userid: **admin** with  password **admin**
 - Set the namespace context to **ace** or **mq** or **eventstreams**, depending on which namespace you want to work with most.
1. Execute `kubectl get pods` for this namespace, or `kubectl -n ace get pods` or `kubectl -n mq get pods` or `kubectl -n eventstreams get pods` for each specified namespace.
1. Execute `kubectl describe pod <pod-name>` for each pod that you want to examine. Some possible values for `<pod-name>` are:
  - `orders-ib-92e8-0` for the ace+mq pod (to troubleshoot the **orders** Integration Server and/or the **acemqserver** Queue Manager)
  - `mq-ibm-mq-0` for the standalone mq pod (to troubleshoot the **mq** Queue Manager)

In addition, you can use the UI for troubleshooting:
1. In a browser session, navigate to the ICP Portal: https://mycluster.icp:8443.
1. Using the hamburger menu, navigate to `Workloads` -> `Helm Releases`.
1. Use the `Search releases` box to filter to the Helm Release you want to analyse. (For example **order**, **mq**, **ace**, **es**). Click on the name of the one you want to work with.
1. Inside that Helm Release, find a pod or other artefact that looks promising and click `View Logs`.






### Test the deployed ACE APIs
Now you will test each API, using cURL, before moving on.

1. Once deployed, your ACE Management UI should display both the `inventory` and `order` servers, as shown:
 ![](./images/cipdemo/appconnect.gif)
1. First, test the `inventory` API.
  - In the ACE Management UI, click down to the `inventory` API and write down or copy to the clipboard the unique value for `REST API Base URL` in your environment (eg **inventory.10.0.0.1.nip.io:3xxxx**).:
 ![](./images/cipdemo/appconn_inventory_api.gif)
  - In the Terminal session, test the `inventory` flow, by using cURL to GET the information from the Base URL appended by the `/retrieve` operation, thus:
 ```
 curl -k -X GET http://inventory.10.0.0.1.nip.io:3xxxx/orders/v1/retrieve
 ```
1. Now, test the `orders` API.
  - Back in the ACE Management UI, click into the `orders` server, and then into the `orders` API. You will see something that resembles the following. Write down or copy to the clipboard the unique value for `REST API Base URL` in your environment (eg **orders.10.0.0.1.nip.io:3xxxx**).
  ![](./images/cipdemo/appconn_order_api.gif)
  - Back in the Terminal session, navigate to _/home/student_ and test the `orders` flow, by using cURL to POST the contents of the `order.json` file, to the Base URL appended by the `/create` operation, thus:
 ```
 curl -k -X POST http://orders.10.0.0.1.nip.io:3xxxx/orders/v1/create -d @order.json
 ```


Once you have confirmed the functionality for both `order` and `inventory` message flows, export the swagger for each from the ACE Management Portal into _/home/student_. You will be using these in the next section.

### Review MQ Portion - standalone MQ (Queue Manager **mq**)
To check that your use of cURL has caused the `orders` flow to correctly put messages onto the relevant MQ queue on the remote Queue Manager (**mq**), you can use the MQ Console.

1. Open the MQ Console for the Queue Manager in one of these  ways:
  - Point a browser session at the ICP Main Portal: `https://mycluster.icp:8443`, open `Workloads` -> `Helm releases`, find the Helm Release called `mq` and on the right `Launch` -> `console-https`.
  - Point a browser session at the ICP4I Platform Navigator: `https://mycluster.icp/integration`, and under `Messaging` select `mq`.
  - Point a browser session directly at the MQ Console: `https://mycluster.icp/integration/instance/mq`

1. You should see the MQ Console and the widgets that you added earlier.
1. On the Queues widget, note that the queue depth should be non-zero. After each test, you can refresh this widget using the refresh button, and the queue depth should increase.
 ![](./images/cipdemo/ace-refresh-button.jpg)
1. To see the content of a message, highlight the relevant queue, then click `Browse messages`.
 ![](./images/cipdemo/ace-browse-messages-UI.jpg)

At this point, you have shown the message generated by New Orders has been successfully put a message to the relevant queue. It is expected that a follow-on applications (or flow) will be written to get that message and take further actions.

You have shown that MQ can be used as a mechanism for transmitting message information asynchronously in a point-to-point paradigm, to enable integration between applications.

Note also that the availability of the MQ Console makes examining messages on queues in this remote Queue Manager fairly easy.

### Review MQ Portion - ACE with MQ (Queue Manager **acemqserver**)
To check that your use of cURL has caused the `orders` flow to correctly put messages onto the relevant MQ queue on the local Queue Manager (in the same pod as ACE), perform the following inside a Terminal Window on the Developer Machine (signed in as _student_),

**Note:** that the local Queue Manager (called) **acemqserver**) is in the same pod as the Integration Server, and was created using a Helm Chart "ACE with MQ". Local queue managers created this way are intended for system use only (for internal ACE functions).

For queue managers created in this way there is no MQ Console available, so we recommend you use command line tools to check messages, thus:
1. Sign into the relevant ICP namespace:
 - In the Terminal session, execute `sudo cloudctl login`.
 - Provide the password for student: **Passw0rd!**
 - Ensure that the API Endpoint **https://mycluster.icp:8443** is specified. If a different one is specified, execute `sudo cloudctl logout` and try again.
 - Provide the CIP userid: **admin** with  password **admin**
 - Set the namespace context to **ace**. (You must choose this namespace, because the kubectl work you are about to do is for this namespace.)
1. Execute  `kubectl get pods`. Identify the orders pod (named **orders-ib-xxxx-x**) and note its name.
1. Execute  `kubectl exec <pod-name> dspmq` . You will see **acemqserver** (the Queue Manager) running.
1. Browse messages on the **NEWORDER.MQ** queue that you defined earlier, by executing `kubectl exec -it <pod-name> /opt/mqm/samp/bin/amqsbcg NEWORDER.MQ`
1. (Note: experienced MQ and Linux users could also start a bash shell to the relevant pod as follows: `kubectl exec -it <pod-name> -- /bin/bash` and use your own methods for examining MQ.)

At this point, you have shown the message generated by New Orders has been successfully put a message to the relevant queue. It is expected that a follow-on flow in the same pod will be written to get that message and take further action.

You have shown that MQ can be used as a mechanism for transmitting message information asynchronously in a point-to-point paradigm, to enable integration between applications.

### Review Event Streams portion

To check that your use of cURL has caused the `orders` flow to correctly publish messages to the relevant Event Streams topic, use the Event Streams dashboard as follows:

1. In your browser session (if still open), switch back to the Event Streams dashboard. Or navigate to the Event Streams dashboard as follows:
 - Open a browser session to the Cloud Integration Platform home: `https://mycluster.icp/integration/`
 - Under `Messaging`, select the `es` link, to open the Event Streams Dashboard.
 - (If you see an error screen similar to that shown below, then simply select the `Open es` link in the middle.)
		![](./images/cipdemo/open_es_error.jpeg)
 - Provide or accept the credentials (Username **admin** Password **admin**) , then `Log In`.
1. Select the `Topics` tab.
1. Click on the name of the only topic you should see there: `NewOrders`, to open that topic.
1. Click on the `Messages` tab, to see all messages.
1. Click on a message, and you will see a screen similar to the following:
	  ![](./images/cipdemo/es-show-message.jpg)
1. Note that as you run more tests and publish more messages to Event Streams, you can click `Next offset` to show further messages.

At this point, you have shown the message generated by New Orders has been successfully published to an Event Streams topic. Other applications or flows could be written to subscribe to that message and take further actions.

You have shown that Event Streams can be used as a mechanism for transmitting message information asynchronously in a publish-subscribe paradigm, to enable loose coupling between applications.

## Create API Facades

**Note** at the time of writing this lab, there is an issue with API Connect that must be resolved before continuing, but fortunately is easily fixed.

1. Open the main Admin console for APIC by opening a new browser tab on the Developer machine to `https://mgmt.10.0.0.1.nip.io/admin`
2. login with the credentials of `admin`/`7Ir0n-hide`

Create APIs for each of the inventory, order and AcmeMart APIs.

>**Note:** the Swagger for the two ACE flows can be imported as APIs using the `From Existing Open API Service` option in API Connect.  The AcmeMart swagger can be downloaded from the main developer page and then imported, but use the `New Open API` option instead.

1.  Open up your API Manager Window inside the developer machine - `https://mgmt.mycluster.icp.nip.io/manager`. The login should happen automatically as it will use your ICP credentials for login.
2.  Click on the `Manage Catalogs`
3.  Click on `Sandbox`
4.  Click on `Settings` from left menu
5.  Click on `Gateway Services`
6.  Click `Edit` and choose the `Gateway` service you configured previously and click on `Save`.
7.  From the API Manager homescreen, use the menu on the left to navigate to `Develop`.
8.  On the APIs and Products screen click `Add` -> `Api`
9.  Select the `From Existing OpenAPI Service` radio button option. Click `Next`.
10.  Click the `browse` button to navigate to the `Tutorial_swagger.json` file on your file system you had saved previously.
11.  Click `Next`
12.  Review the settings then click `Next` twice (do not activate the API yet).
13.  Review the summary to ensure the API was created properly, and then click the `Edit API` button.

**For the AcmeMartUtilityAPI** you will need to modify your invoke URL to look like the following:

`http://(yourapp ip):(your port)$(request.path)$(request.search)`

Where `yourapp ip` and `your port` is the port that ICP has put your Utility App.

![](./images/cipdemo/invoke.gif)

14.  Click the `Assemble` tab to bring up the Assembly editor.


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
2. Open a new browser tab and paste the URL to launch the Developer Portal

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
  https://apigw.10.0.0.1.nip.io/admin-admin/sandbox/orders/v1/create \
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





## Analyze API consumption

### Launch the API Manager and navigate to Analytics

1.  Sign on to the API Manager UI of IBM API Connect

2.  Click on the tile entitled `Manage Catalogs`.

3.  Click on the tile representing the catalog you have published the REST facades to (eg. `Sandbox`)

4.  From the catalog configuration screen, hover over the icons on the left, and click on the `Analytics` tab.

![](./images/cipdemo/analytics_tab.jpeg)

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
