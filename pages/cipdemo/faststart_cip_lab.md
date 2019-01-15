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

This lab will be a team effort.  It is highly suggested that you form your teams of folks who can cover the entire I&D Portfolio - especially: App Connect Enterprise (using Toolkit), API Connect, Messaging (MQ and ES).  Its helpful also to have someone who knows `kubectl` commands and has some background with ICP. 

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

* ICP Main Portal running on `https://10.0.0.1:8443`
* CIP Platform Navigator running on: `https://10.0.0.5/icip1-navigator1`

You should be able to access all portals from the Platform Navigator, but if you need to access the URL directly, they are provided below:

* App Connect Enterprise Manager Portal running on `https://ace.10.0.0.5.nip.io/ace-ace`
* API Connect API Manager Portal on `https://mgmt.10.0.0.5.nip.io/manager`
* MQ Portal on `https://10.0.0.5:31694/ibmmq/console/`
* Event Streams on `https://10.0.0.5:32208/gettingstarted?integration=1.0.0`

**Note**: there are some known certificate issue with the various portals on this environment. These are fairly easy to workaround, and will be fixed in a later release of this demo environment.

2. The environment you are using is the same environment used with the CIP Bootcamp.  It consists of 9 different nodes.  8 of which are ICP Nodes and one is a developer image that you will be using to access the ICP User Interfaces.  You can access this VM directly using the Skytap interface to use the X-Windows based components. The biggest difference with this environment vs what you used at the bootcamp is that all of the base CIP Charts are already installed and configured. 

3. **VERY IMPORTANT** It is very important you do not suspend your lab environment.  We have seen cases when the environment goes into suspend mode, the Rook-Ceph shared storage gets corrupted.  Also it is a good practice that you shut down ICP before powering down your enviroment.  A script has been included to handle that for you that will be explained in the coming sections.  When you power down your environment, you can safely use the `power off` option as the shared storage can interfere with the normal graceful shutdown method.  So far using power off hasn't caused any problems that we are aware of.  If you find that your API Connect environments are not coming up properly, it suggested that you run the `./icpStopStart.sh stop` script (you may need to run it more than once).  When it stops, go ahead and run the `./icpStopStart.sh start`.  As indicated previously, this may take ~30 minutes or so to come up.  

3. You are also able to SSH into your environment.  You can find out the exact address to SSH to in your Skytap Environment window under `networking` and `published services`.  The Published Services set up for you will be for the SSH Port (22) under the `CIP Master` node.  The published service will look something like `services-uscentral.skytap.com:10000`.  Your environment will have a different port on it.  You can then SSH to to the machine from your local machine.  

4. Additionally, you can access the machine direct via the Skytap UI.  This is functional, but can be cumbersome to work with as its not easy to copy and paste into and out of Skytap, especially for the non X-Windows based environments. 
 
5. Password-less SSH has also been enabled between the Master node and the other nodes in the environment.  **note** a table with the environment configuration is provided below.  Credentials for each machine are `root`/`Passw0rd!`.   

| VM        | Hostname  | IP Address | # of CPU | RAM    | Disk Space (LOCAL) | Shared Storage | Additional Notes |
|-----------|-----------|------------|----------|--------|--------------------|----------------|------------------|
| Master    | master    | 10.0.0.1   | 12       | 32 GB  | 400 GB             | N/A            |                  |
| Proxy     | proxy     | 10.0.0.5   | 4        | 8 GB   | 20 GB              | N/A            |                  |
| Worker 1  | worker-1  | 10.0.0.2   | 8        | 16 GB  | 400 GB             | Monitor  | Ceph Master      |
| Worker 2  | worker-2  | 10.0.0.3   | 8        | 16 GB  | 400 GB             | 			  | 		           |
| Worker 3  | worker-3  | 10.0.0.4   | 8        | 16 GB  | 400 GB             | 			  | 		            |
| Worker 4  | worker-4  | 10.0.0.7   | 8        | 16 GB  | 400 GB             |                |                  |
| Worker 5  | worker-5  | 10.0.0.8   | 8        | 16 GB  | 400 GB             | Ceph OSD1               |                  |
| Worker 6  | worker-6  | 10.0.0.9   | 8        | 16 GB  | 400 GB             | Ceph OSD2               |                  |
| Developer | developer | 10.0.0.6   |          |        |                    |                |                  |


6. If it is not done already, power up your Environment.  It could take about 5 minutes for all nodes to start up.  The master node takes the longest to come up, so if you can see the login prompt from the Skytap UI on the master node, then you are good to go. 

7. SSH into the Master Node or use the Skytap UI.  In the home directory of root (`/root`) there is a script called `icpStopStart.sh`.  Run this script by typing in `icpStopStart.sh start` Note that it takes around 30 minutes for the ICP Services to come up completely.

8. You can tell if the start of ICP is complete by checking using one of these two methods:
	- Click on the Developer Machine, and it will take you directly to the Developer Machine running X-Windows.  Should you need to Authenticate, you can use the credentials of `student`/`Passw0rd!`. Bring up the Firefox browser.  Navigate to the main ICP UI by going to `https://10.0.0.1:8443`.  The credentials again are `admin/admin`.  If you can log in and see the main ICP Dashboard, you are good to go.  
	- Alternatively, you can SSH to the Master node.  Execute a `cloudctl login` from the command line.  If all there services are up, it will prompt you for credentials (use `admin`/`admin`) and setup your kubernetes environment.
8. The best place to do your Kubernetes CLI work is from the Master node.  Again, Before you can execute any of the `kubectl` commands you will need to execute a `cloudctl login`. 
9. Next step is to find The Platform Navigator UI can be use to create and manage instances of all of the components that make up the Cloud Integration Platform. 
10. You can access the Platform Navigator using the browser on the developer machine.  The URL for the navigator was set up in this environment as: `https://10.0.0.5/cip-platform-navigator`.  You might be asked to authenticate into ICP again, but once you do that you should now see the Platform Navigator page.
11. The Platform Navigator is designed for you to easily keep track of your integration toolset.  Here you can see all of the various APIC, Event Streams, MQ and ACE instances you have running.  You can also add new instances using the Platform Navigator.


### Key Concepts - Troubleshooting / Recovery

There are times where things may not be going right, so your best bet is to use `kubectl` commands to uncover more information about what is going on.  If you post queries in Slack about trouble with the lab, we will be asking you to execute a series of these, depending upon what you were doing, and what error occured.  A list of useful commands are provided below:

 - `kubectl get pods -n <some-namespace>` This command shows all of the pods in a given name space.  Here you will see if any pods are up, down, errored or otherwise in transition.
 - `kubectl describe pods <some-pod> -n <some-namespace>` This will provide verbose information about a given pod.  You can use the `describe` command for other objects.
 - `kubectl logs <some-object> -n <some-namespace>` This command will work with other objects as well.  You can also tail the logs by using the `-f` switch at the end.

 If you happen to mess up an install of the components, its not difficult to recover.  You can use the ICP UI to remove the release or use the CLI by issuing `helm delete --purge <helm-release-name> --tls`.  Be careful using this though, as this process is not reversable.
 
 Logging is also done inside of ICP using the ELK stack.  You can access the logging inside of ICP and use Elastic Search commands to drill into things.  For example, if you were to bring up Kibana and enter in a search like `kubernetes.namespace:<your namespace>`.  Using this along with the `kubectl` commands gives you a lot of power to dig into root cause.
 

Lab Requirements 
-------------------------------------------

## Configure MQ

Configure your MQ on your ICP environment according to these parameters:

## Configure Event Streams

Configure Event Streams on your ICP environment according to these parameters

## ACE Integration Assets 

10. Your integration assets are found in this repository: `https://github.com/ibm-cloudintegration/techguides`.  You can find the specific files you need in the `/techguides/pages/cipdemo` directory.  **hint** clone this on your Developer machine so you don't have to copy the files over.
11. Below is a description of each of the files in archive that you should take note of (disregard the others).

| File                           | Description                                                                                      |
|--------------------------------|--------------------------------------------------------------------------------------------------|
| faststartflows.zip             | ACE Project Interchange export of integration flows                                                     |
| inventoryproject.generated.bar | generated bar file for the Inventory API.  You will be deploying this as is into the environment |
| order.json                     | test data for the order api                                                                      |
| orderproject.generated.bar     | original bar file for order API. Disregard this, you will be generating a new bar file to deploy |

keep the project interchange zip file handy, you will be loading that up into the toolkit in a later section.


## AcmeMart Microservices

You will need to download the AcmeMart microservices and deploy the containers on ICP.  


14. Access the `master` node via SSH session.  Make a new directory in the root home directory, call it whatever you like.  Change directories into that.
13. Once in that new directory, clone this repository here on the `master` node:  `https://github.com/dashby3000/AcmeMartUtilityAPI`.  The reason we do this on the master node because we need to load the docker images into ICP.
14. go into the AcmeMartUtilityAPI directory.  Also, now is a good time to make sure you are logged into your ICP environment - execute a `cloudctl login` using the `admin`/`admin` credentials.  Make sure you are in the `default` name space.
15. run this command:  `docker build . -t acmemartutilityapi`
16. Tag your new image by running this command:  `docker tag acmemartutilityapi:v1.0.0 mycluster.icp:8500/acmemartapi/acmemartutilityapi:v1.0.0`
17. Push the docker image out to your ICP instance by issuing this command `docker push mycluster.icp:8500/acmemartapi/acmemartutilityapi:v1.0.0`.
18. Create a new namespace in the ICP instance.  Easiest way to do that is via the ICP UI.  On the Developer Machine, direct your browser to `https://10.0.0.1:8443`.  Login using the credentials of `admin`/`admin`.
14. In the UI, starting from the top left hamburger icon select `Manage` -> `Namespaces`.  Create a new namespace and call it `acmemartapi`.  Using the `ibm-anyuid-hostpath-psp` security policy is fine for this one.
18. Next step is to deploy the microservices into ICP.  Create a new Deployment in ICP using the UI via Hamburger Icon in top left. Go to `Workloads -> Deployments`.  Click `+Create Deployment`.
19. In the `General` tab.  Give it a name of `acmemart`.  Select the new namespace created previous via the dropdown (`acmemartapi`). Leave Replicas at `1`.
20. Go to `Container Settings`.  Set the name to `acmemartutility`.  Set the `Image` value to `mycluster.icp:8500/acmemartapi/acmemartutilityapi:v1.0.0`

	![](./images/cipdemo/deployment2.gif)

21. In the same tab, scroll down to where the TCP ports are.  We need to open up a few ports on this image such that it can interact with all components.  Set the ports to the following

	- TCP - 3000
	- TCP - 9093
	- TCP - 443

22. Click `Create`.  The new pod should be created very quickly.  You click on your release and view the results in the UI. **Note** you will not see a `Launch` button like you see in the screenshot until you complete the Service in the next section.

	![](./images/cipdemo/deployment_done.gif)
	
23. Once you see the Pod up, the next step is to bind a Network service that can be associated with the Pod.
24. From the Hamburger menu on top left go to `Workloads` -> `Services`.
25. Create a New Service.  Call it `acmemartapp`.
26. Select the proper namespace - `acmemartapi`
27. Under `Type` select from the dropdown `NodePort`

	![](./images/cipdemo/service3.gif)

28. Under the `Labels` tab set the Label to `app` and value to `acmemart`.
29. Under the `Ports` tab. Create 3 ports per the chart below:

| Protocol  | Name         | Port | Target Port |
|-----------|--------------|------|-------------|
| TCP       | app          | 3000 | 3000        |
| TCP       | eventstreams | 9093 | 9093        |
| TCP       | kafka        | 443  | 443         |


29. Ports should look like the following when done:

	![](./images/cipdemo/serviceports.gif)

30. Under the `Selectors` tab.  Set the `Selector` to `app` and the Value to `acmemart`.
30. Click Create and the service should create quickly. 
31. You will know the Service was generated properly when you return back to your deployment, and you see the `Launch` button in the upper right hand corner.  

	![](./images/cipdemo/service_deployment_done_launch.gif)

32. Click the Launch button and from the dropdown, select the `app` button and it should launch the main portal for the AcmeMartUtils. If so, then your microservices for the demo has deployed successfully. 
33. The AcmeMart microservices comes with its own swagger based developer documentation.  From the main App screen, click on the `Developer Docs`.

	![](./images/cipdemo/acedemo.gif)

34. There are 3 categories of APIs here.  Click on the `Utility` apis.  From the list Select, the `GET /Utilities/ping` option
35. Find the `try it out` button.  Click it.  It should return back the date/time.

	![](./images/cipdemo/pingtest.gif)

## Modify the Order ACE Flow

Import the project interchange provided in the `faststartflows.zip` file.

Modify the `Order` flow by adding two additional operations. after the App Connect REST operation.  One that will put to a MQ queue, and second that will put to a EventStreams Topic.  No transformation required - a vanilla put into each of the endpoints.  Use the environment information provided above.

Generate a new bar files for the orders flow.

## Deploy the bar files

Deploy the `inventoryproject.generated.bar` as provided to the CIP environment.

Deploy the newly generated bar file for the orders flow.

**Hint** each will need to be done separately.  Also create unique hostnames for each flow in the `NodePort IP` setting when configuring the Helm Release. e.g. `orders.10.0.0.5.nip.io`

Be sure to test using cURL before moving to next step.

Use the following value as input for the inventory API `key` query parameter : `AJ1-05.jpg` 

Once you have confirmed the functionality for both `order` and `inventory` message flows, export the swagger for each from the ACE Management Portal.  Save it to the file system on the developer machine, you will be using this in the next section

Once Deployed, your ACE Management UI should display both `inventory` and `order` APIs running

![](./images/cipdemo/appconnect.gif)


## Create API Facades

Create APIs for each of the inventory, order and AcmeMart APIs.

**Note** the Swagger for the two ACE flows can be imported as APIs using the `From Existing Open API Service` option in API Connect.  The AcmeMart swagger can be downloaded from the main developer page and then imported, but use the `New Open API` option instead


## Test the entire flow

We will be using cURL to test the entire flow of the assets deployed today.  This is going to simulate what the mobile app would be doing at demo runtime by executing a series of steps in sequence





### Conclusion
You have completed the initial CIP Labs by taking a base ICP install and then imported some basic integration assets and exposed that as an API running on the configuration.  The key value is having the single platform that runs all of the capability that can be managed easily, user the power of Kubernetes.  

 
**End of Lab**
