---
title: CIP Bootcamp
toc: false
sidebar: labs_sidebar
folder: pots/cip-bootcamp
permalink: /cip-bootcamp-lab.html
summary: Introduction to the Cloud Integration Platform
applies_to: [developer,administrator,consumer]
---

# Getting Started
===============

## Familiarization with the Cloud Integration Platform

The Cloud Integration Platform (CIP) is built on top of IBM Cloud Private.  This lab guide is meant to give you an overview of the capabilities of the platform, and get some hands on time with setting up your own environment for the purpose of creating your own demo environment.

This lab guide assumes some familiarity with IBM Cloud Private, Docker & Kubernetes, as well as Helm.  

There are two primary components of CIP:

**IBM Cloud Private (ICP)** -  An application platform for developing and managing on-premises, containerized applications. It is an integrated environment for managing containers that includes the container orchestrator Kubernetes, a private image repository, a management console, and monitoring frameworks. In this particular boot camp, ICP is deployed on Softlayer, using Skytap as the primary operational interface.

**Platform Navigator** -- is a application that is built to run on top of IBM Cloud Private that provides a primary user interface where instances of App Connect Enterprise (ACE), API Connect (APIC), Event Streams and MQ Advanced can be managed from a single place.  It is automatically deployed with the CIP Bundle.

As a convention for these labs, a <span style="color:red"> **red box** </span> will be used to identify a particular area, and when information is to be entered and/or an action is to be taken, it will be in **bold** characters. <span style="color:red"> **Red arrows or lines** </span> may be used to indicate specific items that need to be noted as part of the lab

A number of sections will be labeled “Key Idea” or “Key Concept”. Be sure to review these sections when you come across them, as they will  explain important items as it relates to the lab content.

Lab Overview
-------------------------------------------

This lab is broken up into two key parts. 

**Part One**:  Create and Deploy your CIP Applications

**Part Two**:  Deploy Integration Assets


Part one contains the hands on section of the lab where you will get acquainted with the Platform Navigator and be setting up the individual components of the platform.

Part two you will be deploying some basic assets to your newly created environment


Part One: Create and Deploy your CIP Applications
-------------------------------------------

1.  You have been provided a pre-installed environment of the Cloud Integration Platform.  It includes a vanilla ICP 3.1.1 install with the CIP Platform Navigator installed and all of the base container images that make up CIP.
2. The environment you are using consists of 9 different nodes.  8 of which are ICP Nodes and one is a developer image that you will be using to access the ICP User Interfaces.  You can access this VM directly using the Skytap interface.
3. **VERY IMPORTANT** It is very important you do not suspend your lab environment.  We have seen cases when the environment goes into suspend mode, the Rook-Ceph shared storage gets corrupted.  Also it is a good practice that you shut down ICP before powering down your enviroment.  A script has been included to handle that for you that will be explained in the coming sections.
3. You are also able to SSH into your environment.  You can find out the exact address to SSH to in your Skytap Environment window under *networking* and `published services`.  The Published Service set up for you will be for the SSH Port (22) under the `CIP Master` node.  The published service will look something like `services-uscentral.skytap.com:10000`.  Your environment will have a different port on it.  You can then SSH to to the machine from your local machine.  
4. Additionally, you can access the machine direct via the Skytap UI.  This is functional, but can be cumbersome to work with as its not easy to copy and paste into Skytap.  
5. Password-less SSH has also been enabled between the Master node and the other nodes in the environment.  **note** a table with the environment configuration is provided below.  Credentials for each machine are `root`/`Passw0rd`.   You shouldn't need this for the lab, but its provided for your information.

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
7. SSH into the Master Node or use the Skytap UI.  In the home directory of root (`/root`) there is a script called `icpStopStart.sh`.  Run this script by typing in `icpStopStart.sh start` Note that it takes 30 minutes for the ICP Services to come up completely on startup.
8. You can tell if the start of icp is complete by checking using one of these two methods:
	- Click on the Developer Machine, and it will take you directly to the Developer Machine running X-Windows.  Should you need to Authenticate, you can use the credentials of `student`/`Passw0rd!`. Bring up the Firefox browser.  Navigate to the main ICP UI by going to `https://10.0.0.1:8443`.  The credentials again are `admin/admin`.  If you can log in and see the main ICP Dashboard, you are good to go.  
	- Alternatively, you can SSH to the Master node.  Execute a `cloudctl login` from the command line.  If all there services are up, it will prompt you for credentials (use `admin`/`admin`) and setup your kubernetes environment.
8. The best place to do your kubernetes CLI work is from the Master node.  Again, Before you can execute any of the `kubectl` commands you will need to execute a `cloudctl login`. 
9. Next step is to find The Platform Navigator UI can be use to create and manage instances of all of the components that make up the Cloud Integration Platform. **note** at the time of this lab, the ability to create and manage Aspera instances has not yet be added to the Platform Navigator, and will be added at a later date.
10. You can access the Platform Navigator using the browser on the developer machine.  The URL for the navigator was set up in this environment as: `https://10.0.0.5/icip1-navigator1`.  You might be asked to authenticate into ICP again, but once you do that you should now see the Platform Navigator page.
11. The Platform Navigator is designed for you to easily keep track of your integration toolset.  Here you can see all of the various APIC, Event Streams, MQ and ACE instances you have running.  You can also add new instances using the Platform Navigator.
11. Each application instance requires its own namespace and pull secret.  To support this lab, namespaces and pull secrets have been provided for you already, with the exception of Event Streams (ES).  We'll talk more about ES later in the lab.


### App Connect Enterprise

10. Let's start by creating an App Connect Enterprise Integration Server.  From the Platform Navigator, locate the middle panel for `Application Integration` and click on the `add new instance` link.
11. A pop up window will give you some important information about pre-requisites for creating the instance.  Click `continue`.
12. This will take you to the chart configuration.  If you scroll to the bottom, you can peruse the information about the chart you are about to use to create your instance of App Connect Enteprise.  Click the `Configure` button to configure your chart.
13. Here you can fil in the information for install of your chart.  Give your helm release a `name`.  Call the release `ace-icip`.
14. To the right of the `name` field, Select the `ace` namespace from the `Target Namespace` dropdown.
15. Tick checkbox under `license`
16. Expand the `All Parameters` twisty
17. Find the `Image Pull Secret` line.  Set that to `acekey` (This is pre-defined for you, normally, you would need to create this).
18. For the `Hostname of the ingress proxy to be configured` section. Change the default value provided to `mycluster.icp`.  
18. **IMPORTANT** - Find the `Enable Persistant Storage` Section.  This box must be unticked (disabled).  In order to support persistant storage, Gluster storage must be used (Ceph is not supported with ACE).  Using Gluster Storage beyond the scope of this lab.
19. Leave the other default values in the chart.  Scroll down and Click `Install` button on lower right portion of the screen.
20. Once you see the `Installation Started` pop up. You can validate the progress via the `Helm Releases` in the ICP Portal or via the command line.
21. Return to your command line and  open up a SSH session to your master node on your laptop or desktop.  **Note** Each environment has a Skytap Published Service set up for every node on port 22 so you can SSH to the nodes from your machine.  If you do a `kubectl get pods -n ace` you should see your pods in a running state.
22. Returning to the Platform Navigator, you should see your new ACE instance there.  Click on it to bring up the ACE Enterprise UI.  **Note**  if you encounter issues with the self signed certs on the browser.  Click on the link where it says `open ace`.  This will open a new browser window and should allow you to access the UI properly.
22. The ACE Dashboard can alternatively be accessed via a web browser. Run the following commands to get the dashboard URL via SSH:

`export ACE_DASHBOARD_URL=$(kubectl get ingress ace-icip-ibm-ace-dashboard-icip-prod-hostname -o jsonpath="{.spec.rules[0].host}{.spec.rules[0].http.paths[0].path}")`

then `echo "Open your web browser to https://${ACE_DASHBOARD_URL}"`

your output will look something like:  `Open your web browser to https://mycluster.icp/ace-ace-icip`

*Hint* If the first command fails, you might not be in the `ace` name space.  Quickest way to set your context is just to relogin using `cloudctl login` and then set your namespace to `ace`.  

23. If you can see the App Connect Enterprise portal, then you are are done with this step.  We'll load in a .bar file later.

## MQ

10. From the Platform Navigator, locate the panel on the far right for `Messaging` and click on the `add new instance` link and then select `New Message Queue` instance.
11. A pop up window will give you some important information about pre-requisites for creating the instance.  Click `continue`.
12. This will take you to the chart configuration.  If you scroll to the bottom, you can peruse the information about the chart you are about to use to create your instance of MQ.  Click the `Configure` button to configure your chart.
13. Here you can fil in the information for install of your chart.  Give your helm release a `name`.  Call the release `mq-icip`.
14. To the right of the `name` field, Select the `acemq` namespace from the `Target Namespace` dropdown.
15. Tick checkbox under `license`
16. Expand the `All Parameters` twisty
17. Untick the `Production Usage` tick box.
18. Under `Persistence` untick both `Enable Persistence` and `Use Dynamic Provisioning`.  *Note* failure to untick this will cause your install to fail.
19. Under the `Queue Manager` section.  Enter your queue manager name  `QMGR.DEV`.
19. You can leave the other settings as defaults.
20. Click `Install` to start the process.
21. You can monitor the installation from Helm Releases in the ICP UI or via the command line using `kubectl get pods -n acemq`.  Once you see your pod up and running, you can navigate to the UI via the Platform Navigator.  Find your MQ instance and click on it.  **Note**  if you encounter issues with the self signed certs in the browser.  Click on the link where it says `open mq`.  This will open a new browser window and should allow you to access the MQ UI properly.
22. In the MQ UI, click on the `3 dots` icon to the right of the properties Hamburger icon.  Select the option that says `Add a new dashboard tab` this creates a new management tab for your Queue Manager
23. Click on the `+` button to create a new queue.  Call this Queue `TEST.OUT`.
24. Authority records??
24. You Have now configured MQ


## Event Streams


Add in some Integration Assets
-----------------------------

1.  While logged into your App Connect Instance, in the main dashboard, Click on the **New+** button and then select **Flows for an API**.


	![](./images/pots/ace-designer/acplab1newflow.png)



### Conclusion
You have completed the "Hello World" section of the App Connect Designer labs.  You should have a good feel for the UI now, and the subsequent labs will dive into some more complex use cases that are based on real world examples.

 
**End of Lab**
