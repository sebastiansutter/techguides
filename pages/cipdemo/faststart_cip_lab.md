---
title: European BP ICP4I enablement
toc: false
sidebar: labs_sidebar
folder: pots/cipdemo
permalink: /cip-demo-lab.html
summary: Deep dive into IBM Cloud Pak for Integration
applies_to: [developer,administrator,consumer]
---

# Exploring the IBM Cloud Pak for Integration
----------------

## Overview

The goal of these lab sessions is to provide you some hands-on experience with the IBM Cloud Private platform and the IBM Cloud Pak for Integration.

This lab guide will lay out the scenario for you, along with the requirements of what you are to complete.  It will then provide generally a step-by-step walkthrough of what to complete (although in some places it will not be quite as formulaic).

You are not expected to be an expert in any of the Integration portfolio, but some knowledge of each of the components used in this lab will help. They are
- IBM App Connect Enterprise (ACE, including using the ACE Toolkit)
- IBM API Connect
- IBM MQ
- IBM Event Streams).

It will also help to have some knowledge of `kubectl` and `docker` commands, and  some background with IBM Cloud Private or Kubernetes.


### Lab Overview

You will be building a demo environment that supports the "AcmeMart" demo scenario.  There are several components that make up this demo, and the key areas in scope for this lab are as follows:

*  AcmeMart, an application based on Node.js, comprising several microservices
*  Asynchronous and publish/subscribe messaging, provided by MQ and Event Streams
*  Two REST APIs that interact with IBM Cloud-based APIs, provided by integration flows within App Connect Enterprise (ACE)
*  API Facades for the ACE APIs, provided by API Connect

All the already-created assets, as well as assets that you will create, will be implemented on the IBM Cloud Private environment provided for you.  All prerequisites and utilities should already be installed on the environments provided.  If there are other utilities and tools you would like to use to support you during this exercise, you may do so at your own risk.

The environment in which you are working is completely standalone, so your work is not visible to other students. Equally, any errors or misconfigurations on the part of other students cannot affect your environment.


### Lab Environment Overview

You have been provided with a pre-installed environment of IBM Cloud Private with the IBM Cloud Pak for Integration (previously called the Cloud Integration Platform). Henceforth these names are shortened thus:

* IBM Cloud Private = ICP
* IBM Cloud Pak for Integration = ICP4I

The base charts have already been deployed and configured.  This includes specifically:

* The ICP Main Portal running on `https://mycluster.icp:8443`
* The ICP4I Platform Navigator running on: `https://mycluster.icp/integration`

You will normally access all of the individual components' portals from the ICP4I Platform Navigator, but if you want to access the URLs directly, we list them here for you:

* IBM MQ Portal on `https://mycluster.icp:31681/ibmmq/console/`
* IBM Event Streams on `https://mycluster.icp/integration/instance/es`
* IBM App Connect Enterprise Portal running on `https://mycluster.icp/integration/instance/ace1`
* IBM API Connect API Manager Portal on `https://mgmt.mycluster.icp.nip.io/manager`

This lab's Skytap environment  consists of 6 different nodes,  5 of which are ICP Nodes. The 6th is a Developer Image, that you will use to perform nearly all of your work using command line and Firefox browser.

A table with the Skytap environment configuration is provided below.  Credentials for each machine are `root`/`Passw0rd!`.

| VM        | Hostname  | IP Address | # of CPU | RAM    | Disk Space (LOCAL) | Additional Notes |
|-----------|-----------|------------|----------|--------|--------------------|------------------|
| Master    | master    | 10.0.0.1   | 24       | 39 GB  | 425 GB             | hostname mycluster.icp|
| Worker 2  | worker-2  | 10.0.0.3   | 12        | 47 GB  | 600 GB             |You should not need to sign on to this|
| Worker 3  | worker-3  | 10.0.0.4   | 12        | 47 GB  | 600 GB            		  |You should not need to sign on to this|
| Worker 4  | worker-4  | 10.0.0.5   | 12        | 47 GB  | 600 GB             		  |You should not need to sign on to this|
| Worker 5  | worker-5  | 10.0.0.6   | 12        | 47 GB  | 500 GB                            |You should not need to sign on to this|
| Developer | developer | 10.0.0.9   |          |        |                                  |You will do almost all your work using this one|



**VERY IMPORTANT** It is very important you do not suspend your lab environment.  When the environment goes into suspend mode, it is likely that the Rook-Ceph shared storage will get corrupted.

it is a good practice to shut down ICP before powering down your environment.  A script has been included to do this - it is `./icpStopStart` and you should run it from the Master Node when signed on as **root**.
- Having run `./icpStopStart stop`, you can  power down the Skytap environment safely using the `Power off` option. The shared storage can interfere with the normal "graceful shutdown" method.  We are not aware of `Power off` causing any problems.
- If you find that your components (eg ACE or MQ or ES or API Connect) are not starting properly, and you have 30+ minutes to spare, you can execute `./icpStopStart.sh stop` from the Master Node when signed on as `root`.  When this script finishes, execute `./icpStopStart.sh start`.  Note that it can take around 30 minutes for ICP and all the ICP4I components to start completely.

You can access the Master Node and the Developer Image directly via the sessions provided from the Skytap UI.  This is functional, but it is not intuitive to copy and paste into and out of Skytap, especially for the non X-Windows based environments.

As well as using the sessions provided by the Skytap UI, you could also SSH into the Master Node and the Developer Image.
 - You can find  the addresses for SSH in your Skytap Environment window under `Networking` and `Published services`.
 -The Published Services set up for you will be for the SSH Port (22) under the `CIP Master` node.  The published service will look something like `services-uscentral.skytap.com:10000`.
 - You can then SSH to the relevant image.


Password-less SSH has also been enabled between the Master Node and the other nodes in the environment.

## Start-Up Instructions

1. If it is not done already, power up your Skytap environment.  It could take about 5 minutes for all nodes to start up.  The Master Node typically takes the longest to start, so if you can see, from the Skytap UI, the login prompt on the Master Node, then you can start to use the environment.
1. The ICP infrastructure and all of the ICP4I components are configured to start when the environment starts. It typically takes around 30 minutes for all the components to be ready.
8. You can tell if the ICP infrastructure is ready by going to the ICM Main Portal thus:
	- On the Developer Image, start a Firefox browser, and navigate to the  ICP Main Portal at `https://mycluster.icp:8443`.  If you can log in using the credentials `admin/admin` and see the ICP Dashboard, then the ICP infrastructure is ready.
1. You can also tell if the ICP infrastructure is ready by trying to log into it from a command line, thus:
 - On the Master Node, start a Terminal session and execute `sudo cloudctl login`.
 - Provide the password for student: **Passw0rd!**.
  - Ensure that the API Endpoint **https://mycluster.icp:8443** is specified (if a different API Endpoint is specified, execute `sudo cloudctl logout` and try again).
 - Provide the ICP4I userID **admin** and password **admin**
 - When prompted, choose any namespace.
 - If all the above works, then the ICP infrastructure is ready.
8. The best place to do your Kubernetes CLI work is from the Master Node. As shown above, before you can execute any of the `kubectl` commands you will need to execute `sudo cloudctl login` - this signs you in to the ICP infrastructure.

Now you will access the ICP4I Platform Navigator. This is the UI that allows you to  create and manage instances of all of the components that make up ICP4I (in this lab: ACE, MQ, Event Streams and API Connect).
1. On the Developer Image, start a Firefox browser, and navigate to the ICP4I Platform Navigator at `https://mycluster.icp/integration`.  To authenticate, use the ICP4I userID `admin` and password `admin`.
1. The Platform Navigator is designed for you to easily keep track of your integration components and artefacts.  Here you can see all of the various API Connect, Event Streams, MQ, Aspera and ACE instances you have running.  You can also use the Platform Navigator to add new instances.


### Key Concepts - Troubleshooting / Recovery

There are times where things may not be going right. It is often productive to use `kubectl` commands to perform problem determination. A list of useful commands is provided here:

 - `kubectl get pods -n <some-namespace>` This shows all of the pods in a given namespace.  You can determine if any pods are running, in error or otherwise in transition.
 - `kubectl describe pods <some-pod> -n <some-namespace>` This provides verbose information about a given pod.  Note: you can also use the `describe` command for other objects.
 - `kubectl logs <some-object> -n <some-namespace>` This command will work with other objects as well.  You can also tail the logs by using the `-f` switch at the end.

 Logging is also done inside ICP using the ELK stack.  You can access the logging inside  ICP and use Elastic Search commands to drill into information.  An example Kibana search is `kubernetes.namespace:<your namespace>`.  Using this along with the `kubectl` commands gives you a lot of power to dig into root causes.

You can also the ICP Portal to help with problem determination. Use the hamburger menu and then options such as `Workloads` -> `Helm Releases` or `Workloads` -> `Deployments`.


## ACE Integration Assets

1. The ACE integration assets have already been populated into this lab for you. (You can also find them in this repository: `https://github.com/ibm-cloudintegration/techguides/pages/cipdemo`.)
2. Below is a description of each of the files that are relevant to the ACE portion of this lab.

| File                           | Description                                                                                      |
|--------------------------------|--------------------------------------------------------------------------------------------------|
| ACEflows.zip             | an ACE Project Interchange export of all the integration flows - allowing you to restore the code for the supplied ACE flows and APIs, in case anything goes wrong ! |
| orders.json             | a sample order file |
| generatesecret.sh     | a script file that generates an ICP Secret, for use when deploying BAR files (Helm Charts) into ICP|
| serverconf.yaml     | skeleton file, used by generatesecret.sh|
| setdbparms.txt     | parameter file, used by generatesecret.sh, which you will edit|
| truststorePassword.txt     | parameter file, used by generatesecret.sh - it holds the password for the truststore that you will generate |
| mqsc.txt     | parameter file, used by generatesecret.sh - it holds the MQSC command CREATE QLOCAL(NEWORDER.MQ)|




## AcmeMart Microservices

In this section, you  will  download the AcmeMart microservices and deploy the containers on ICP.


1. On the Master Node, start a Terminal session; you will be logged in as `student`.
1. Change to root thus `su - root` (providing the password for student: **Passw0rd!**).
1. In the `/root` directory, make a new directory, calling it whatever you like.  Change directories down into that new one.
2. In that new directory, clone this repository here on the `master` node:  `https://github.com/asimX/FS-CIP-Microservices`.  You do this on the Master Node because you need to load the docker images into ICP.
1. You might need to authenticate using your github.com account.  If you don't have one, you can sign up for your free one here at `github.com`
3. Go into the FS-CIP-Microservices directory.
3. Log into the ICP environment, thus:
 - Execute `sudo cloudctl login`.
 - Provide the password for student: **Passw0rd!**.
  - Ensure that the API Endpoint **https://mycluster.icp:8443** is specified (if a different API Endpoint is specified, execute `sudo cloudctl logout` and try again)
	- Provide the ICP4I userID: **admin** with  password **admin**
1. When prompted, choose the namespace context  **default**.

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
1. On the Developer Machine, navigate to the Event Streams dashboard thus:
 - Open a browser session to the ICP4I Platform Navigator: `https://mycluster.icp/integration/`
 - Under `Messaging`, select the `es` link, to open the Event Streams Dashboard.
 - (If you see an error screen similar to that shown below, then simply select the `Open es` link in the middle.)
		![](./images/cipdemo/open_es_error.jpeg)
 - Provide or accept the credentials (Username **admin** Password **admin**) , then `Log In`.

1. Now you are in the main Event Streams dashboard. Select the `Topics` tab and then `Create Topic`, to start creating a new topic:
 - You will be taken through 4 screens where you specify basic Topic configuration. Note the` Advanced` option : this would present all the parameters on one long screen. You can explore the Advanced options if you like, but for this lab we assume you will just specify the basic configuration parameters.
  - For the `Topic name`, we recommend **NewOrder**. This is the topic to which, in this lab, a message will be published by ACE, each time a new order is created. If any application wants to subscribe to this message then they must be told what this value is.
 - Leave the `Partitions` as **1**. (For scalability in Production, you might specify **3** or more.)
 - You can specify whatever `Message Retention` you like; we recommend leaving it as **A week**.
 - The `Replicas` setting is used for High Availability; we recommend you leave it to default.
  - Select `Create Topic` at the end. You will be taken back to the Topics screen.

Now you will create and extract Event Streams connection information, for use later on.
1. Select `Connect to this cluster`, to pull in the window that allows you to create and copy configuration information.

1. Make a note of the value of the `bootstrap server`. (Store it in a temporary file, or just write it down.) Make sure you note it correctly, because you must specify this exactly in the ACE configuration later.
1. Use the correct button to download the `PEM Certificate`. Store it in `/home/student/Downloads`.
		![](./images/cipdemo/bootstrap-server-and-certificate.png)
1. Use the section on the right to create an API key:
 - First `Name your application`. We recommend you specify **ordersFlow** here. This is used, within Event Streams, to report connections and activity, but it is not used further in this lab session.  You could specify ACE-Connection, or ACEorders, or anything else that identifies the connection for you.
 - Specify `Produce, consume and create topics`. Although in this lab session we ask you only to produce messages, specifying this allows you to extend the lab and do more later, if you want. In a Production system you could use this to restrict access.
 - Select the slider, to specify `All topics`. Although in this lab session we ask you to use only the topic you created above,  specifying `All topics` allows you to extend the lab and do more later, if you want. In a Production system you could use this to restrict access.
 - Ensure that the slider specifying `All` consumer groups is enabled. In a Production system you could use this to restrict access.
 - After selecting `Generate API key`, make sure you download the API key (a JSON file called **es-api-key.json**) to _/home/student/Downloads_.
 - Don't forget to select `Create a new API key`, to make it all happen.


### Generate a Secret object

Now you will generate a "Secret" object. This is a Kubernetes construct that lets you store and manage sensitive information, such as passwords and certificates. In this lab, a Secret is used to store connectivity information for connecting securely from an ACE Helm Release (aka Integration Server) to your instance of Event Streams.

(More information on Secrets can be found here:
https://kubernetes.io/docs/concepts/configuration/secret/ )

1. Prepare the PEM file you downloaded earlier.
 - On the Developer Image, open a "Files" session.
 - Move the PEM certificate file that you downloaded from Event Streams earlier (probably called **es-cert.pem**) from _/home/student/Downloads_ to _/home/student/generateSecret_.
 - Rename this PEM certificate file in _/home/student/generateSecret_ to the following: **truststore-escert.crt** (yes, this means that it will no longer be a PEM file). Note: that this name structure must be of the form **truststore-ALIASNAME.crt**, where ALIASNAME will be generated as the alias for the certificate. Note down your specific  ALIASNAME (**escert**), because you will need to specify this later when deploying a BAR file.
1. Go back to _/home/student/Downloads_, open the downloaded JSON file **es-api-key.json** in your favourite editor and copy the API key to the clipboard. Note: copy only the API Key, not the quotes around it.
	![](./images/cipdemo/ace-copy-api-key.jpg)

1. On the Developer Image, open a Terminal session.  Note that you will be signed in as _student_, and be in directory _/home/student/_.
1. Change to working directory  _/home/student/generateSecret_. This directory contains a tool called  _generatesecret.sh_, which will generate the Secret you need. It also contains some files that form input into that tool. This directory now also has your renamed PEM file (**truststore-escert.crt**) in it, which also forms input into the tool.
1. Use either `gedit setdbparms.txt` or `vi setdbparms.txt` to edit the **_setdbparms.txt_** file.
    - Replace the characters `<over-write with API Key>` with the API key that you copied to the clipboard a moment ago. Note that this must be done accurately, otherwise the connection from ACE to Event Streams will not work. The content of the file should look like this:
 ```
 kafka::KAFKA token TYu9y5iTqBxKeCDuko-BBCnaMpDn2zIZYJp7uYUzGFWh
 IntSvr::truststorePass thisispwdfortruststore password
 ```
    - The first line means "when ACE uses its Kafka client to connect, use this API key as the password". (**token** is the userID for that password, but it is not used.)
    - The second line means "when ACE refers to the identity _IntSvr::truststorePass_, it will use the password **password**. (**thisispwdfortruststore** is the userID for that password, but it is not used.) Note that in the set of configuration files already in directory _/home/student/generateSecret_, **truststorePassword.txt** has already been created, containing the matching value **password**.
    - `Save` your changes and exit the editor.
1. Take the following steps to sign into the ICP4I namespace and run the tool to generate the Secret:
 - In the Terminal session, execute `sudo cloudctl login`.
 - Provide the password for student: **Passw0rd!**.
 - Ensure that the API Endpoint **https://mycluster.icp:8443** is specified. If a different one is specified, execute `sudo cloudctl logout` and try again.
 - Provide the ICP4I userID: **admin** with  password **admin**
 - Set the namespace context to **ace**. (This is because the Secret we are about to generate must be in namespace **ace**.)
 - Execute `./generatesecret.sh orders-secret`. This script takes in the various configuration files (including the PEM certificate and the API key) and then uses _kubectl_ to generate the Secret with name **orders-secret**. You should see a success message `secret/orders-secret created`.
1. Check that the secret has been created, using the UI.
 - In a browser session, navigate to the ICP Portal: https://mycluster.icp:8443.
 - Using the hamburger menu, navigate to `Configuration` -> `Secrets`.
 - Confirm that Secret **orders-secret** has been created, and click it to open and check that it is in namespace **ace**.

You have now finished preparing Event Streams, and you have created the artefacts needed by ACE  to connect to Event Streams.

## Configuring remote MQ to permit ACE connection

The `orders` API provided by ACE will put one message onto a queue on a local Queue Manager (called **acemqserver**), and also the same message onto a queue on a remote Queue Manager (called **mq**). This lab session uses those two Queue Managers, to   illustrate the configuration differences between a remote and a local Queue Manager.

You do not need to perform any extra configuration on the local Queue Manager **acemqserver**. Because it is in the same pod as the Integration Server, it is a local QMgr and the `orders` API will connect to it using "server bindings" ("server bindings" need no extra configuration on the QMgr.)

In this section, you will perform the necessary configuration on the remote Queue Manager **mq**, to permit ACE to connect to it.

## Configure MQ Helm Chart to include MQ Artefacts
Kubernetes and ICP standards recommend that you define and alter MQ artefacts (queues, channels and so on) as part of the Helm Chart. This means that these changes will be placed inside the Queue Manager **at deployment time**, instead of **after deployment**. The main reason is that if and when Kubernetes restarts pods, it will restore those changes.

You define (and alter) artefacts using MQSC commands stored inside a Secret. The following instructions describe how to do this.
1. On the Developer Machine, duplicate the directory **…/generateSecret** to **…/generateSecret-for-mq**.
1. Inside the new directory, delete the following:
 - **serverconf.yaml**
 - **setdbparms.txt**
1. Leave the **truststorePassword.txt** file, to enable the `generatesecret.sh` command to work. For this lab, it does not matter what this contains because you do not use it, but the `generatesecret.sh` command needs it to exist.
1. Inside the new directory, edit the **mqsc.txt** file. Remove all existing MQSC commands and write new MQSC commands to achieve the following (use your own skills and the MQ Knowledge Center):
 - Alter the Queue Manager properties, to specify that CHLAUTH is **disabled**
 - Define a new local queue called **NEWORDER.MQ**
 - Define a new Server-Connection channel called **ACE.TO.mq**. For MCAUserID, specify **mqm**.
 - Alter the system-provided Authentication Information **SYSTEM.DEFAULT.AUTHINFO.IDPWOS**, to specify Check Client Connections = **NONE** and Check Local Connections = **NONE**
 - **REFRESH SECURITY** for everything
1. Log on to the cloud using `sudo cloudctl login`. Make sure you specify namespace `mq`.
1. Inside the new directory, run the **generatesecret.sh** command, to generate a new secret called **mq-secret**.

1. Check that the secret has been created, using the UI.
  - In a browser session, navigate to the ICP Portal: https://mycluster.icp:8443.
  - Using the hamburger menu, navigate to `Configuration` -> `Secrets`.
  - Confirm that Secret **mq-secret** has been created, and click it to open and check that it is in namespace **mq**.

1. Use the ICP Portal to remove the existing **mq** Helm Release, and go to the ICP4I Platform Navigator to ensure that the **mq** instance has been deleted.

1. From the ICP4I Platform Navigator, add a new Message Queue instance:
 ![](./images/cipdemo/ace-add-new-instance.jpg)
     - Call it **mq**.
     - Specify namespace **mq**
     - Specify the `Secret name` as above (**mq-secret**).
     - Make sure that `Enable Persistence` is **unchecked**.
     - Leave the Queue Manager name blank (it will default to the Helm Release name)
     - Leave all other parameters to default

After this new instance has been deployed, use the Web Console to check its configuration, as follows.

1. Open the MQ Console for the Queue Manager in one of these ways:
  - (Recommended) Point a browser session at the ICP4I Platform Navigator: `https://mycluster.icp/integration`, and under `Messaging` select `mq`.
  - Point a browser session at the ICP Main Portal: `https://mycluster.icp:8443`, open `Workloads` -> `Helm releases`, find the Helm Release called `mq` and on the right `Launch` -> `console-https`.
  - Point a browser session directly at the MQ Console: `https://mycluster.icp/integration/instance/mq`
3. Click the Queue Manager name `mq` to highlight it. (This is the name of the Queue Manager already created in this pod). Select `Properties`.

   ![](./images/cipdemo/ace-mq-properties.jpg)

5. On the `Communication` tab, find the `CHLAUTH Records` property and make sure that it is set to `Disabled`.

   ![](./images/cipdemo/ace-chlauth-disabled.jpg)

1. At the top right, click `Add Widget`, then select your Queue Manager `mq` and select the `Queues` widget.
 - In your new Queues widget, you should see your  queue called **NEWORDER.MQ**, of type  `local`.
1. At the top right, click `Add Widget` again, then select your Queue Manager `mq` and select the `Channels` widget.
 - In your new Channels widget, you should see your channel called **ACE.TO.mq**.
 - Click `ACE.TO.mq` to select it, and then click `Properties`, and verify that the properties match what you specified earlier.



### Create and Configure MQ artefacts post-deployment (not recommended)

The remote Queue Manager called **mq** has already been created and deployed, in its own Helm Chart.

You could now use the MQ Console to perform some configuration. However, those will not be stored in the initial Helm Chart, and so if Kubernetes restarts relevant pods, your changes will be lost.

The following instructions describe how to use the MQ Console to view and change Queue Manager properties and artefacts.
1. Open the MQ Console for the Queue Manager in one of these ways:
  - (Recommended) Point a browser session at the ICP4I Platform Navigator: `https://mycluster.icp/integration`, and under `Messaging` select `mq`.
  - Point a browser session at the ICP Main Portal: `https://mycluster.icp:8443`, open `Workloads` -> `Helm releases`, find the Helm Release called `mq` and on the right `Launch` -> `console-https`.
  - Point a browser session directly at the MQ Console: `https://mycluster.icp/integration/instance/mq`
3. Click the Queue Manager name `mq` to highlight it. (This is the name of the Queue Manager already created in this pod). Select `Properties`.

   ![](./images/cipdemo/ace-mq-properties.jpg)

5. On the `Communication` tab, find the `CHLAUTH Records` property and make it `Disabled`.

   ![](./images/cipdemo/ace-chlauth-disabled.jpg)

1. Don't forget to `Save` and then `Close`.

You will now add a new queue and a new channel. You will also change the MQ authentication information.

1. At the top right, click `Add Widget`, then select your Queue Manager `mq` and select the `Queues` widget.
  -  In your new Queues widget, click on `Create (+)` to create a new queue called **NEWORDER.MQ**, of type  `local`.
  - You have just created a queue, onto which messages can be put.
1. At the top right, click `Add Widget` again, then select your Queue Manager `mq` and select the `Channels` widget.
  -  In your new Channels widget, click on `Create (+)` to create a new channel called **ACE.TO.mq**, of type `Server-connection`.
  - Having created it, click `ACE.TO.mq` to select it, and then click `Properties`.
  - On the MCA tab, for `MCA User ID` specify **mqm**. This forces this userID to be used when a connection is attempted. For this lab session it makes for an easy connection; in a Production environment more strict security should be applied.
	![](./images/cipdemo/ace-specify-MCA.jpg)
  - You have just created a channel, which will be used by the MQ Client built into ACE Integration Server when its flows want to connect to this Queue Manager.
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

### Confirm MQ IP Port
Finally, you will check which port the MQ Listener is listening on, thus:

1. Within the MQ Console, add a new `Listeners` widget.
  -  In your new Listeners widget, click on the cogwheel to configure the widget, and select `System objects` -> `Show`.
	![](./images/cipdemo/ace-authinfo-cogwheel.jpg)
  - You should see the system-provided Listener called `SYSTEM.LISTENER.TCP.1`. Make a mental note of the port for this listener (almost certainly it will be the MQ default of **1414**).
	  ![](./images/cipdemo/ace-showing-listener.jpg)
1. In a browser session, navigate to the ICP Portal: https://mycluster.icp:8443.
  - Using the hamburger menu, navigate to `Workloads` -> `Helm releases`.
  - Click the `mq` release to open it.
  - Towards the bottom, you should see the Service called `mq-ibm-mq`, and under `PORT(S)` you should see the port mentally noted earlier (probably **1414**).
  - Write down the associated mapped port number, which will be **3xxxx**, as shown below:

    ![](./images/cipdemo/ace-determine-mq-listener-port.jpg)

You have now identified that the queue manager `mq` is listening on port **1414** within its pod, and that this is mapped to the external port **3xxxx**. The latter number (**3xxxx**) is the one that you will need to specify when connecting to this Queue Manager.

You have now finished preparing the remote Queue Manager `mq`, to allow the MQ Client within ACE to connect to it.


## Modify the `orders` API within ACE

The REST APIs within ACE, that form part of the overall solution, are mostly already written for you. You will now make changes to one of those REST APIs (namely the `orders` API) to make it put messages on MQ queues and publish a message to an Event Streams topic).

1. On the Developer Image, go back to the Terminal session and navigate to directory _/home/student_.
1. Start the Ace Toolkit by executing `sudo ./ace-11.0.0.3/ace toolkit`. Note that the ACE Toolkit may start as a tiny window on the screen. Use the cursor to grab the corner of this screen and expand it.

	![](./images/cipdemo/ace-tiny-toolkit-start.jpg)

1. Make sure that you specify the correct workspace _/home/student/IBM/workspace_ as shown:

    ![](./images/cipdemo/ace-select-workspace.jpg)

1. You should see, in the Application Development window on the lefthand side, three REST APIs and their artefacts:
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

These are the detailed instructions for modifying the ACE subflow.

1. Drill down `orders` -> `Resources` -> `Subflows` and open `New_Order.subflow`.
1. From the ACE palette, drag and drop an `HTTPHeader` node, two `MQOutput` nodes and a `KafkaProducer` node. Position them and connect them as shown in the screenshot above.
1. Select the `Http Header` node. Configure its properties thus:
 - On the `HTTPResponse` tab, select **Delete HTTP Header**.

   ![](./images/cipdemo/ace-delete-httpheader.jpg)

1. Select the first `MQOutput` node. Configure its properties thus:
  - For `Queue Name` specify **NEWORDER.MQ**. This is the queue onto which the message will be put. Note: in this instance we are hard-coding this queue name; typically it will be parameterised (for ACE specialists: this parameterisation uses _LocalEnvironment.Destination.MQ.DestinationData.queueName_).
  - For `Connection` select `Local queue manager`, because this is connection to the local Queue Manager (running inside the same pod that is also running ACE).
  - For `Destination queue manager` name specify **acemqserver** (case sensitive). This is the name of the local Queue Manager, which will be defined when this flow and its Integration Server and Queue manager are deployed . Note: in this lab we are hard-coding this Queue Manager name; typically it will be parameterised (using an MQ Policy).

   ![](./images/cipdemo/ace-mqoutput.jpg)

2. Select the second `MQOutput1` node. Configure its properties thus:
	 - For `Queue Name` specify **NEWORDER.MQ**. This is the queue onto which the message will be put. Note: in this instance we are hard-coding this queue name; typically it will be parameterised (for ACE specialists: this parameterisation uses _LocalEnvironment.Destination.MQ.DestinationData.queueName_).
	 - In the `MQ Connection` tab, specify the following MQ Client connection properties. This is a connection to a remote Queue Manager, running on a separate pod in the cluster. Note: in this lab we are hard-coding these connection details; typically it will be parameterised (using an MQ Policy).
      - `Connection`: **MQ client connection properties**
      - `Destination queue manager name`: **mq** (case-sensitive) - this is the name of the remote Queue Manager
      - `Queue manager host name`: **10.0.0.1** - this is the access IP address for the relevant pod
      - `Listener port number`: **3xxxx** - this must match the mapped port number that you determined earlier by looking at the Helm Release
      - `Channel name`: **ACE.TO.mq** (case-sensitive)- this must match the channel name that you created earlier by using the MQ Console

   ![](./images/cipdemo/ace-mqoutput1.jpg)

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

The BAR file containing the changed `orders` API is now ready for deployment. You called it **orders.bar**, and it resides in the ACE workspace (which is _/home/student/IBM/workspace/BARfiles_).


## Deploy the `orders` BAR File

In this section you will deploy the `orders` BAR file (the BAR File you changed immediately above) to the ICP4I environment.

Deployment of a BAR file includes the creation of the Integration Server in which it will run, and you do this by configuring and deploying a Helm Chart. This Helm Chart includes all the details of the Integration Server, as well as the BAR file itself.

In a DevOps environment, you would expect to configure and deploy the Helm Chart programmatically. In a Development environment, such as this lab, the typical way to deploy a BAR File (and create the Integration Server) is to start from the ACE Dashboard.

>_**Note:**_ When deploying, you will create a unique hostname for each BAR file; this is referred to as the `NodePort IP` setting when configuring the Helm Release. For example: **orders.10.0.0.1.nip.io** and **inventory.10.0.0.1.nip.io**. The **10.0.0.1.nip.io** portion specifies that an on-demand service (**nip.io**) is used to route to **10.0.0.1** (the IP address of the ICP proxy node). The whole value of `NodePort` must be unique; it points to the HTTP listener for the specific Integration Server.

1. Use one of the following methods to get to the ACE Dashboard.
 - Either go directly by opening a browser session to **https://mycluster.icp/ace-ace1**
 - Or start with the ICP4I Platform Navigator: **https://mycluster.icp/integration/**, and select `ace1`. Note: if this appears to fail as shown in the following screenshot, simply select `Open ace1` on the failure screen and it should work.

	  ![](./images/cipdemo/open_ace_error.jpeg)


2. On the ACE Dashboard, make sure you are on the Servers tab.

3. Follow these steps **_very carefully_**, to deploy the `orders` flow that you have just changed.
	- From the ACE Dashboard, in the `Servers` tab or in the `Integration` tab, select `Add server+` to start the process.
	- Browse to _/home/student/IBM/workspace/BARfiles_, select the BAR file `orders.bar`, and `Continue`.
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
	- For the `Image pull secret` specify **sa-ace**. This Secret was created when the Cloud Pak for Integration was installed, and it contains the credentials for ICP to access its private docker repository.
	- Untick `Enable persistence`. This is because provisioning volumes for persistence in this particular lab environment is sometimes troublesome. For a lab environment, persistence is not required for the MQ Queue Manager. In a Production environment, you would probably enable persistence.
	- For the `NodePort`, we recommend **orders.10.0.0.1.nip.io**. We recommend that the first part of the name (**orders** in this case) is identical to the `Integration Server name`, because there is a 1 to 1 relationship here.
	- For the `Queue manager name`, specify **acemqserver**. This defines the name that ICP will give to the associated local queue manager (which matches what we specified in the `MQOutput` node earlier).
	- For the `Certificate alias name`, specify **escert**. This is used by the Integration Server when it connects to Event Streams. You defined the value **escert** earlier, when you created the Secret.
	- Note that you could also specify the `Integration Server name`. However, we recommend that you leave this blank, so that the Helm Chart name (**orders** in this case) is used.

		![](./images/cipdemo/ace_hc_5.png)
		![](./images/cipdemo/ace_hc_6b.png )
		![](./images/cipdemo/ace_hc_8.png)
		![](./images/cipdemo/ace-disable-persistence.jpg)
		![](./images/cipdemo/ace_hc_9.png)
		![](./images/cipdemo/ace_hc_10.png)

	- Leave the remaining settings as defaults and then click `Install` at the bottom.
	- You should now see the "Installation started" window, and your Helm Chart will now start to install.

		![](./images/cipdemo/ace-installation-started.jpg)

	> **Tip:** when you see the "Installation started" window, click on the little cross at the  top right to cancel only that window (see screenshot above). This will mean that your Helm Chart, with all the configuration you have just typed in, will remain in the browser. In turn, this means that, if something goes wrong during installation, you can a) easily check what parameters you typed and b) easily try the installation again.

4. Note: if you don't see the "Installation started" window, and the deployment seems not to have started, then scroll to the top of the screen and you may see an error message.

	![](./images/cipdemo/ace-ttext-error.png)

	If the error message is `t.text is not a function` (as shown above) then take the following steps to remove the `helm-api` pods. NB You could also use the ICP Console to do all of the above, if you feel more comfortable taking that route.
	- In a Terminal session, execute `sudo cloudctl login`.
	- Provide the password for student: **Passw0rd!**
	- Ensure that the API Endpoint **https://mycluster.icp:8443** is specified. If a different one is specified, execute `sudo cloudctl logout` and try again.
	- Provide the ICP4I userID: **admin** with  password **admin**
	- Set the namespace context to **kube-system**.
	- Execute `kubectl get pods | grep helm-api` to find all the offending pods.
	- For each of the pods that starts with **helm-api**, execute `kubectl delete pod <pod-name>`
	- Wait for a minute or so, to allow ICP to recreate a fresh `helm-api` pod.
	- Then try re-deploying again.


5. Return to the ACE Dashboard and confirm that the `orders` Integration Server has been correctly deployed. You should wait a few seconds to a handful of minutes, for the deployment to succeed fully. Use the `Refresh` button.

### Some help for ACE, Event Streams and MQ Problem Determination
If the deployment does not seem to succeed, use kubectl to perform problem determination, as follows:

1. Sign into the relevant ICP namespace:
 - In the Terminal session, execute `sudo cloudctl login`.
 - Provide the password for student: **Passw0rd!**
 - Ensure that the API Endpoint **https://mycluster.icp:8443** is specified. If a different one is specified, execute `sudo cloudctl logout` and try again.
 - Provide the ICP4I userID: **admin** with  password **admin**
 - Set the namespace context to **ace** or **mq** or **eventstreams**, depending on which namespace you want to work with most.
1. Execute `kubectl get pods` for this namespace, or `kubectl -n ace get pods` or `kubectl -n mq get pods` or `kubectl -n eventstreams get pods` for each specified namespace.
1. Execute `kubectl logs <pod-name>` or `kubectl logs <pod-name> -n <namespace>` to see the logs for a pod.
1. Execute `kubectl describe pod <pod-name>` for each pod that you want to examine. Some possible values for `<pod-name>` are:
  - `orders-ib-92e8-0` for the ace+mq pod (to troubleshoot the **orders** Integration Server and/or the **acemqserver** Queue Manager)
  - `mq-ibm-mq-0` for the standalone mq pod (to troubleshoot the **mq** Queue Manager)

In addition, you can use the UI for troubleshooting:
1. In a browser session, navigate to the ICP Portal: https://mycluster.icp:8443.
1. Using the hamburger menu, navigate to `Workloads` -> `Helm Releases`.
1. Use the `Search releases` box to filter to the Helm Release you want to analyse. (For example **order**, **mq**, **ace**, **es**). Click on the name of the one you want to work with.
1. Inside that Helm Release, find a pod or other artefact that looks promising and click `View Logs`.






## Test the deployed ACE APIs
Now you will test each API, using cURL, before moving on.

Note that in this section, you will be testing the APIs as exposed by ACE - the **inventory** one that was already deployed and the **orders** one that you have just modified and deployed. (Don't let the use of the word "API" confuse you - you are not yet touching the API Connect component of ICP4I.)

1. Once deployed, your ACE Management UI should display both the `inventory` and `order` servers, as shown:
 ![](./images/cipdemo/appconnect.gif)
1. First, test the `inventory` API.
  - In the ACE Management UI, click down to the `inventory` API and write down or copy to the clipboard the unique value for `REST API Base URL` in your environment (eg **inventory.10.0.0.1.nip.io:3xxxx**).:
 ![](./images/cipdemo/appconn_inventory_api.gif)
  - In the Terminal session, test the `inventory` flow, by using cURL to GET the information from the Base URL appended by the `/retrieve` operation, and specifying **46DA2599-CB6F-4FCC-A75F-D0E491AB6769.jpg** as the **key**, thus:
 ```
curl -k http://inventory.10.0.0.1.nip.io:32041/storeinventory/v1/retrieve?key=46DA2599-CB6F-4FCC-A75F-D0E491AB6769.jpg
 ```
(Apologies for the very long key.)
  - You should see a very long response mainly in Latin (!) that starts thus:
```
{"Inventory":[{"color":"ash grey color","location":"In Store","pictureFile":"AJ1-01","productDescription":"Lorem ipsum dolor ...
```
and ends thus:
```
... finibus tortor.","productID":"AJ1-10","productName":"Red, White & Metallic Blue","qtyOnHand":"1100","rating":"1","type":"AirJordan1","typeDescription":"Air Jordan 1 (So Crispy)","unitPrice":"110.99"}]}
```
1. Now, test the `orders` API.
  - Back in the ACE Management UI, click into the `orders` server, and then into the `orders` API. You will see something that resembles the following. Write down or copy to the clipboard the unique value for `REST API Base URL` in your environment (eg **orders.10.0.0.1.nip.io:3xxxx**).
  ![](./images/cipdemo/appconn_order_api.gif)
  - Back in the Terminal session, navigate to _/home/student_ and test the `orders` flow, by using cURL to POST the contents of the `orders.json` file, to the Base URL appended by the `/create` operation, thus:
 ```
 curl -k -X POST http://orders.10.0.0.1.nip.io:3xxxx/orders/v1/create -d @orders.json
 ```
  - You should see a short response that looks like this. (Note: the flow will also have put messages onto MQ queues and published a message to an Event Streams topic; you will check those in a moment.)
```
{"accountid":"A-10000","orderid":"1632603"}
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

At this point, you have shown that the Orders API has successfully put a message to the relevant queue on a remote Queue Manager. It is expected that a follow-on applications (or flow) will be written to get that message and take further actions.

You have shown that MQ can be used as a mechanism for transmitting message information asynchronously in a point-to-point paradigm, to enable integration between applications.

Note also that the availability of the MQ Console makes examining messages on queues in this remote Queue Manager fairly easy.

### Review MQ Portion - ACE with MQ (Queue Manager **acemqserver**)
To check that your use of cURL has caused the `orders` flow to correctly put messages onto the relevant MQ queue on the local Queue Manager (in the same pod as ACE), perform the following inside a Terminal Window on the Developer Image (signed in as _student_),

**Note:** that the local Queue Manager (called) **acemqserver**) is in the same pod as the Integration Server, and was created using a Helm Chart "ACE with MQ". Local queue managers created this way are intended for system use only (for internal ACE functions).

For queue managers created in this way there is no MQ Console available, so we recommend you use command line tools to check messages, thus:
1. Sign into the relevant ICP namespace:
 - In the Terminal session, execute `sudo cloudctl login`.
 - Provide the password for student: **Passw0rd!**
 - Ensure that the API Endpoint **https://mycluster.icp:8443** is specified. If a different one is specified, execute `sudo cloudctl logout` and try again.
 - Provide the ICP4I userID: **admin** with  password **admin**
 - Set the namespace context to **ace**. (You must choose this namespace, because the kubectl work you are about to do is for this namespace.)
1. Execute  `kubectl get pods`. Identify the orders pod (named **orders-ib-xxxx-x**) and note its name.
1. Execute  `kubectl exec <pod-name> dspmq` . You will see **acemqserver** (the Queue Manager) running.
1. Browse messages on the **NEWORDER.MQ** queue that you defined earlier, by executing `kubectl exec -it <pod-name> /opt/mqm/samp/bin/amqsbcg NEWORDER.MQ`
1. (Note: experienced MQ and Linux users could also start a bash shell to the relevant pod as follows: `kubectl exec -it <pod-name> -- /bin/bash` and use your own methods for examining MQ.)

At this point, you have shown that the Orders API has successfully put a message to the relevant queue on a local Queue Manager. It is expected that a follow-on flow in the same pod will be written to get that message and take further action.

You have shown that MQ can be used as a mechanism for transmitting message information asynchronously in a point-to-point paradigm, to enable integration between applications.

### Review Event Streams portion

To check that your use of cURL has caused the `orders` flow to correctly publish messages to the relevant Event Streams topic, use the Event Streams dashboard as follows:

1. In your browser session (if still open), switch back to the Event Streams dashboard. Or navigate to the Event Streams dashboard as follows:
 - Open a browser session to the ICP4I Platform Navigator: `https://mycluster.icp/integration/`
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



**Note:** We recommend you to use Google Chrome to access the API Connect WebUIs during the lab

Create APIs for each of the inventory, order and AcmeMart APIs.

**Note:** the Swagger for the two ACE flows can be imported as APIs using the `From Existing Open API Service` option in API Connect. The AcmeMart swagger can be downloaded from the main developer page and then imported.

1.  Open up your API Manager Window inside the Developer Image - `https://mgmt.10.0.0.1.nip.io/manager`. The login should happen automatically as it will use your ICP credentials for login.
2.  Click on the `Manage Catalogs`
3.  Click on `Sandbox`
4.  Click on `Settings` from left menu
5.  Click on `Gateway Services`
6.  Ensure there is a gateway service assosiated with the Catalog
![](./images/cipdemo/gatewayservices.jpg)
7.  Click `Back`. From the API Manager homescreen, use the menu on the left to navigate to `Develop`.
8.  On the APIs and Products screen browse over the `AcmeMartUtility` API
9.  Click on the icon on the right, then click `Save as a new version`

![](./images/cipdemo/newversion.jpg)

10. Type in version `1.0.1`

![](./images/cipdemo/version101.jpg)

11.  Select the new version of the API and click on it
12.  Review the API configuration using the `Design` tab
13.  Then click on the `Source` tab

![](./images/cipdemo/apidefinition.jpg)

14.  Review the API definition to ensure the API was created properly, and then click the `Assemble` button on the top.

**For the AcmeMartUtilityAPI** ensure your invoke URL to looks like the following:

`http://(yourapp ip):(your port)$(request.path)$(request.search)`

Where `yourapp ip` and `your port` is the port that ICP has put your Utility App.

![](./images/cipdemo/invoke.gif)

14.  Click the `Play` button on the top to get to the built in test tool

15. Activate your API by clicking on the `Activate API` button

![](./images/cipdemo/activateapi.jpg)

Test your APIs in the test tool inside of API Connect.

![](./images/cipdemo/test_api.gif)

If you see a CORS error, click on the link provided by the test tool and accept the certificate manually, as shown below:

![](./images/cipdemo/cors.jpg)

For the `AcmeMartUtilityAPI`, the quickest way to do this is open up the Assembly view of the AcmeMart and select the `/Utilities/Ping` API. It requires no arguments.

Before:

![](./images/cipdemo/test_acmemart_api1.gif)

After:

![](./images/cipdemo/test_acmemart_api2.gif)


16. You may want to click on `Repeat` to invoke the API multiple times from the test tool

![](./images/cipdemo/repeat.jpg)

Testing the `Inventory` API can be done in the Assembly view by using the following value as input for `key`: `AJ1-05.jpg`

![](./images/cipdemo/store_inventory_assembly_test.gif)

The orders flow can't be (easily) tested inside of the Assembly test view as it requires a json payload.  We will test this using the flow below.

Explore the assembly view and the policy palette on the left.


15. Create an API Product. Go to `Develop`, `Add`, `Product`, `New Product`:

![](./images/cipdemo/createproduct.jpg)

16. Add your APIs to the Product, then click `Next`:

![](./images/cipdemo/addapis.jpg)

17. The `Default` plan is automatically added, click `Next`

18. Publish the product

>**Note:** You can control the product visibility and subscribability at this step

![](./images/cipdemo/publishproduct.jpg)


## Test the entire flow

In this part, you will explore the consumer experience for APIs that have been exposed using API Connect. Using the Developer Portal, you will log in and subscribe to the API Product and test the APIs from the portal, before testing it in a live Web Application.

This part will show the following:
+ How to subscribe to a plan in order to consume an API.
+ How to test an API from the developer portal.
+ How to consume an API from a sample test application.

1. Launch the Developer Portal from your bookmarks if you have the link saved, otherwise you  can obtain the Developer Portal URL from the API Manager. Go to your Catalog (eg. `Sandbox`), from the `Settings` menu select `Portal` to show the configuration. Copy the URL.
![](./images/cipdemo/portalurl.jpg)

2. Open a new browser tab and paste the URL to launch the Developer Portal (admin credentials for the Developer Portal are `admin/Passw0rd!`, developer credentials are `developer/Password!`). We will be using the `developer` account in the next steps

![](./images/cipdemo/portal.png)

3. Click the `API Products` link
4. Log in with the Developer account for your portal. Remember that this account is different from the credentials you use to log in to API Management
5. Click the `API Products` link after logging in
6. Select the API product you published in the previous part and click `Subscribe`
7. The App called `Mobile App` was already created for you. However feel free to create a new App if you want
8. As the result, the application is now subscribed to the product and its credentials are registered to use the Product APIs.

![](./images/cipdemo/appsubscription.jpg)

>In this section, you will use the Developer Portal to test the entire flow you created. This is useful for application developers to try the APIs before their application is fully developed or to simply see the expected response based on inputs they provide the API.

Go back to the `API Product` page, click on the product you published in the previous section
1. Click the logistics AcmeMartUtilityAPI 1.0.0 link on the product page

![](./images/cipdemo/selectapi.jpg)

2. Click the GET /Sneakers/identity method of the API path on the left-hand navigation menu

3. Open `Environment Vesion Info` file located at the desktop and copy the Sample Data for Inventory API key from it
4. Ensure you're using the app the app from the previous section to inkove the API:
5. Paste the key to the `key` field:
![](./images/cipdemo/portaltest.jpg)

Example Output:

![](./images/cipdemo/testoutput.jpg)

Feel free to test other APIs and API methods using the Portal








### Conclusion
You have put together the main building blocks for the combined ICP4I Demonstration.  The only things left are to plug in the mobile app to make these api calls, and make a minor change to the microservices to use the on-premises based event streams versus the cloud. Good luck with the IBM Cloud Pak for Integration!



We appreciate your feedback, please take the enablement [survey](http://ibm.biz/survey1)


**End of Lab**
