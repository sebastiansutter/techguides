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

### Key Concepts - Troubleshooting with `kubectl`

There are times where things may not be going right, so your best bet is to use `kubectl` commands to uncover more information about what is going on.  If you post queries in Slack about trouble with the lab, we will be asking you to execute a series of these, depending upon what you were doing, and what error occured.  A list of useful commands are provided below:

 - `kubectl get pods -n <some-namespace>` This command shows all of the pods in a given name space.  Here you will see if any pods are up, down, errored or otherwise in transition.
 - `kubectl describe pods <some-pod> -n <some-namespace>` This will provide verbose information about a given pod.  You can use the `describe` command for other objects.
 - `kubectl logs <some-object> -n <some-namespace>` This command will work with other objects as well

 There are plenty other commands to use, but these are by far the most common the author of these labs has used while setting these up :)


## App Connect Enterprise

10. Let's start by creating an App Connect Enterprise Integration Server.  From the Platform Navigator, locate the middle panel for `Application Integration` and click on the `add new instance` link.
11. A pop up window will give you some important information about pre-requisites for creating the instance.  Click `continue`.
12. This will take you to the chart configuration.  If you scroll to the bottom, you can peruse the information about the chart you are about to use to create your instance of App Connect Enteprise.  Click the `Configure` button to configure your chart.  Other than the items listed below, we will be accepting the default values in the chart.
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

If you can see the App Connect Enterprise portal, then you are are done with this step.  We'll load in a .bar file later.

## MQ

10. From the Platform Navigator, locate the panel on the far right for `Messaging` and click on the `add new instance` link and then select `New Message Queue` instance.
11. A pop up window will give you some important information about pre-requisites for creating the instance.  Click `continue`.
12. This will take you to the chart configuration.  If you scroll to the bottom, you can peruse the information about the chart you are about to use to create your instance of MQ.  Click the `Configure` button to configure your chart.  Other than the items listed below, we will be accepting the default values in the chart.
13. Here you can fil in the information for install of your chart.  Give your helm release a `name`.  Call the release `mq-icip`.
14. To the right of the `name` field, Select the `acemq` namespace from the `Target Namespace` dropdown.
15. Tick checkbox under `license`
16. Expand the `All Parameters` twisty
17. Untick the `Production Usage` tick box.
18. Under the `Image Pull Secret` section enter in `acemqkey`.
18. Under `Persistence` untick both `Enable Persistence` and `Use Dynamic Provisioning`.  *Note* failure to untick this will cause your install to fail.
19. Under the `Queue Manager` section.  Enter your queue manager name  `QMGR.DEV`.
19. You can leave the other settings as defaults.
20. Click `Install` to start the process.
21. You can monitor the installation from Helm Releases in the ICP UI or via the command line using `kubectl get pods -n acemq`.  Once you see your pod up and running, you can navigate to the UI via the Platform Navigator.  Find your MQ instance and click on it.  **Note**  if you encounter issues with the self signed certs in the browser.  Click on the link where it says `open mq`.  This will open a new browser window and should allow you to access the MQ UI properly.
22. In the MQ UI, click on the `3 dots` icon to the right of the properties Hamburger icon.  Select the option that says `Add a new dashboard tab` this creates a new management tab for your new Queue Manager
23. Click on the `+` button to create a new queue.  Call this Queue `TEST.OUT`.
24. Authority records??
24. You Have now configured MQ


## Event Streams

20. Before we create a new Event Streams instance, you will need to delete the existing Event Streams instance on the environment as well as deleting the existing eventstreams namespace.
21. There a couple of ways to remove an installation.  We'll do this using in two parts, one by UI, one by CLI. Using the ICP UI (`https://10.0.0.1:8443`) go to the top left hamburger config menu select `Workloads` -> `Helm Releases`.  Locate the `eventstreams` install.    Find the `action` icon on the far right (looks like 3 dots).  Click that and then select `Delete`.  ICP will remove all of the Pods that are part of this install.
22. Next we will remove the old namespace. Lets do that via the command line.  Connect to your master node via SSH.  Login via `cloudctl login` if you have to.  Select default namespace.
23. Delete the namespace as follows  `kubectl delete namespace eventstreams`.  Deleting the namespace should be fairly quick.
24. Now lets recreate the namespace.  Do this via `kubectl create namespace eventstreams` 
22. Change context to this new namespace by issuing `kubectl config set-context mycluster-context --user=admin --namespace=eventstreams`
23. re-create your pullsecret then by issuing: `kubectl create secret docker-registry eskey --docker-server=mycluster.icp:8500 --docker-username=admin --docker-password=admin --docker-email=admin`
24. Return back to the Developer Machine via Skytap.  Bring up the Platform Navigator again.  Locate the panel on the far right for `Messaging` and click on the `add new instance` link and then select `New Event Streams` instance.
25. Similar to the other instances you have setup, you will have a pop up that gives you some advice on requirements.  Click `continue`
26. Select the `configuration` tab on the top. Here we will configure the chart for install.  Other than the items listed below, we will be accepting the default values in the chart.
27. Under `Helm Releases` call it `eventstreams-cip`
28. Set the `Target Namespace` to `eventstreams`
29. Tick the checkbox under `License`
30. Moving down, set the `Image Pull Secret` to `eskey`
31. Click the `All Parameters` Twisty
32. Under the Global Install Parameters heading, untick the `Used in Production` checkbox.
33. Modify the `Docker Image Registry` entry by removing the last trailing '/' character.  e.g. `mycluster.icp:8500/icip`
34. Moving down, make sure all of the persistant storage checkboxes are unticked.
35. That is the last change.  Click the `Install` button on the lower right.
36. Check the progress of the install using the command line (via SSH) to the master node.  `kubectl get pods -n eventstreams`
37. If you see all the pods up properly then you are good to go.  
38. You can access the management interface by returning back to the ICP UI (https://10.0.0.1:8443) on the Developer Machine.  From the main ICP UI, upper left hand corner, click on the hamburger icon and go to `Workloads` -> `Helm Releases`.  Locate your Eventstreams release and then click the `launch` button on the far right. From the drop down list select `admin-ui-https`.  If you see the event streams UI come up, you are then ready to configure event streams

## API Connect

20. To install API Connect, there are a few items that are needed in order to properly configure your ICP environment.  These are listed below
	- Persistant Storage (Ceph is required, and has been set up for you already)
	- 7 FQDN's for the various UIs/Portal of API Connect
21. Below is a list of the FQDN's you will be using for this configuration.

### API Connect Management ###
**Platform API Endpoint:**   `https://mgmt.10.0.0.5.nip.io`

**Consumer API Endpoint:**   `https://mgmt.10.0.0.5.nip.io`

**Cloud Admin UI Endpoint:** `https://mgmt.10.0.0.5.nip.io`

**API Manager UI Endpoint:** `https://mgmt.10.0.0.5.nip.io`

### API Connect Portal ###
**Portal Director API Endpoint:** `https://pd.10.0.0.5.nip.io`

**Portal Web UI Endpoint:**       `https://pw.10.0.0.5.nip.io`

### API Connect Analytics ###
**Analytics Ingestion API Endpoint:** `https://ai.10.0.0.5.nip.io`

**Analytics Client API Endpoint:**    `https://ac.10.0.0.5.nip.io`

### API Connect Gateway ###
**Gateway Service API Endpoint:** `https://gws.10.0.0.5.nip.io`

**API Gateway Endpoint:**         `https://apigw.10.0.0.5.nip.io`

21. Return to the Developer Machine and bring up the Platform Navigator.  On the left hand side of the screen locate the `API Lifecycle and Secure Access` section.  At the bottom of this section, locate and click on the `add new instance` link.
21. A pop up will come up, giving you some information about setting up the instance.  Click `Continue`.
22. Click on the `Configuration` heading.  Other than the items listed below, we will be accepting the default values in the chart.
23. Set the `Helm Release` to `apic-cip`.  
24. Moving to the right, set the `Target Namespace` from the dropdown to `apic`
25. Tick the checkbox under `License`
26. Under the `Parameters` heading.  Set the `Registry Secret` to `apickey`.
27. Set `Storage Class` to `rbd-storage-class` This is your Ceph Storage class name. **NOTE** it is very important that this setting is correct, otherwise the setup of the shared storage volumes will fail.
28. Set the `Helm TLS Secret` to `helm-tls-secret`
29. Contuining on, under the `Management` heading.  Here you need to fill in the four different management headings as per the chart in step 2 above.
30. Same thing for the `Portal`, `Analytics`and `Gateway` sections.  Follow the chart above and enter in the values for each of the above values in the FQDN list in Step 2.
31. When done, click the `All Parameters` twisty.
32. Scroll up a bit and start with the values under the `Global` heading.  Select the `Mode` drop down, and set the value to `dev`.
33. Scroll down a bit and located the `Cassandra` heading.  Set that value of `Cluster Size` to the value of `1`.
34. Scroll down some more to the `Gateway` section.  Set the `Replica Count` to the value of `1`.
35. That completes the chart configuration.  Click the `Install` button to start the process.  The Entire Process to install API Connect takes about 15 minutes to spin up all of the pods.  You can monitor things using a command like `watch kubectl get pods -n apic`.  

Add in some Integration Assets
-----------------------------

1.  While logged into your App Connect Instance, in the main dashboard, Click on the **New+** button and then select **Flows for an API**.


	![](./images/pots/ace-designer/acplab1newflow.png)



### Conclusion
You have completed the "Hello World" section of the App Connect Designer labs.  You should have a good feel for the UI now, and the subsequent labs will dive into some more complex use cases that are based on real world examples.

 
**End of Lab**
