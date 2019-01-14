---
title: CIP Bootcamp
toc: false
sidebar: labs_sidebar
folder: pots/cip-bootcamp
permalink: /cip-bootcamp-lab.html
summary: Introduction to the Cloud Integration Platform
applies_to: [developer,administrator,consumer]
---

# Fast Start 2019 Lab - Cloud Integration Platform (CIP)
===============

## Overview

During the CIP Bootcamp, you had the opportunity to get some hands on with the CIP Platform.  This exercise here is to provide you some more hands on with the platform and build your skills with the platform with the goal of creating a demo environment everyone is able to build and use on their own.

This lab guide will lay out the scenario for you, along with the requirements of what you are to complete.  This guide is different than other labs, such that it will not be a step by step walkthrough of what to complete, rather will give you a list of requirements to complete, and it is up to you and your team to deliver on those.

This lab guide assumes some familiarity with the Cloud Integration Platform (CIP), IBM Cloud Private (ICP), Docker & Kubernetes, as well as Helm.  

Lab Overview
-------------------------------------------

You will be building a demo environment that supports the "AcmeMart" demo scenario.  There are several components that make up this demo, but the key areas in scope for this lab are as follows:

* 	Two App Connect Enterprise flows that interact with IBM Cloud based APIs
*  AcmeMart Node.js based APIs
*  On-premises based MQ and ES based assets
*  API Facades for RESTful assets to be created in API Connect

All assets created are to be implemented on the given CIP environment provided for you.  All pre-requisites and utilities should be on the environments provided for you.  If there are other utilities and tools you would like to use to support you during this exercise, you may do so at your own risk.


Lab Environment Overview 
-------------------------------------------

You have been provided a pre-installed environment of the Cloud Integration Platform with the base charts for CIP already deployed and configured.  This includes specifically:

* ICP Main Portal running on `https://10.0.0.1:8443`
* CIP Platform Navigator running on: `https://10.0.0.5/cip-platform-navigator`

You should be able to access all portals from the Platform Navigator, but if you need to access the URL directly, they are provided below:

* App Connect Enterprise Manager Portal running on `https://ace.10.0.0.5.nip.io/ace-ace`
* API Connect API Manager Portal on `https://mgmt.10.0.0.5.nip.io/manager`
* MQ Portal on `https://10.0.0.5:31694/ibmmq/console/`
* Event Streams on `https://10.0.0.5:32208/gettingstarted?integration=1.0.0`

2. The environment you are using consists of 9 different nodes.  8 of which are ICP Nodes and one is a developer image that you will be using to access the ICP User Interfaces.  You can access this VM directly using the Skytap interface to use the X-Windows based components.  

3. **VERY IMPORTANT** It is very important you do not suspend your lab environment.  We have seen cases when the environment goes into suspend mode, the Rook-Ceph shared storage gets corrupted.  Also it is a good practice that you shut down ICP before powering down your enviroment.  A script has been included to handle that for you that will be explained in the coming sections.  When you power down your environment, you can safely use the `power off` option as the shared storage can interfere with the normal graceful shutdown method.  So far using power off hasn't caused any problems that we are aware of.

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

## ACE Integraton Assets 

10. You can clone your repository


## Event Streams

20. Before we create a new Event Streams instance, you will need to delete the existing Event Streams instance and namespace on the environment.


## Reference Material - API Connect Endpoint configuration

### API Connect Management ###
**Platform API Endpoint:**   `mgmt.10.0.0.5.nip.io`

**Consumer API Endpoint:**   `mgmt.10.0.0.5.nip.io`

**Cloud Admin UI Endpoint:** `mgmt.10.0.0.5.nip.io`

**API Manager UI Endpoint:** `mgmt.10.0.0.5.nip.io`

### API Connect Portal ###
**Portal Director API Endpoint:** `pd.10.0.0.5.nip.io`

**Portal Web UI Endpoint:**       `pw.10.0.0.5.nip.io`

### API Connect Analytics ###
**Analytics Ingestion API Endpoint:** `ai.10.0.0.5.nip.io`

**Analytics Client API Endpoint:**    `ac.10.0.0.5.nip.io`

### API Connect Gateway ###
**Gateway Service API Endpoint:** aka Management Endpoint `gws.10.0.0.5.nip.io`

**API Gateway Endpoint:** aka API Invocation Endpoint       `apigw.10.0.0.5.nip.io`

21. Return to the Developer Machine and bring up the Platform Navigator.  On the left hand side of the screen locate the `API Lifecycle and Secure Access` section.  At the bottom of this section, locate and click on the `add new instance` link.
21. A pop up will come up, giving you some information about setting up the instance.  Click `Continue`.
22. Click on the `Configuration` heading.  Other than the items listed below, we will be accepting the default values in the chart.
23. Set the `Helm Release` to `apic-cip`.  
24. Moving to the right, set the `Target Namespace` from the dropdown to `apic`
25. Tick the checkbox under `License`
26. Under the `Parameters` heading.  Set the `Registry Secret` to `apickey`.
27. Set `Storage Class` to `rbd-storage-class` This is your Ceph Storage class name. **NOTE** it is very important that this setting is correct, otherwise the setup of the shared storage volumes will fail.  **Note** you can issue a `kubectl get pvc -n apic` once you kick off the install of your chart to see how the shared space is being used by APIC and if there are any problems e.g. if you see an entry in a `pending` state for an extended period of time.
28. Set the `Helm TLS Secret` to `helm-tls-secret`
29. Contuining on, under the `Management` heading.  Here you need to fill in the four different management headings as per the chart in step 2 above.
30. Continue with the same process for the `Portal`, `Analytics` and `Gateway` sections.  Follow the chart above and enter in the values for each of the above values in the FQDN list in Step 2.
31. When done, click the `All Parameters` twisty.
32. Scroll up a bit and start with the values under the `Global` heading.  Select the `Mode` drop down, and set the value to `dev`.
33. Scroll down a bit and located the `Cassandra` heading.  Set that value of `Cluster Size` to the value of `1`.
34. Scroll down some more to the `Gateway` section.  Set the `Replica Count` to the value of `1`.
35. That completes the chart configuration.  Click the `Install` button to start the process.  The Entire Process to install API Connect takes about 15 minutes to spin up all of the pods.  You can monitor things using the watch command e.g. `watch kubectl get pods -n apic`.
  

## API Connect Configuration

20. These next steps are the standard setup steps for configuring a provider organization inside of APIC.  
21. On the Developer Machine, open a new browser tab to `https://mgmt.10.0.0.5.nip.io/admin`.  Once the Admin Portal comes up, you can log in with `admin/7iron-hide`.  You will be logged in and forced to change your password.  Set it to whatever you like, but don't forget it.
22. You will now be placed into the Cloud Manager Portal.  Click the link to `Configure Topology`
23. Click on `Register Service`
24. Select `Datapower Gateway v5 Compatible`
25. Under Gateway details - give the gateway a name and then under `Management Endpoint` set that value to `https://gws.10.0.0.5.nip.io`
26. Set the `API Invocation endpoint` to ` https://apigw.10.0.0.5.nip.io`.  Click `Save`
28. Once returned back to the Topology screen, click `Register Service` and then select `Analytics`.
29. Set the Title to whatever you like and then change the `management endpoint` to `https://ac.10.0.0.5.nip.io`.  Click `Save`.
30. Once returned back to Topology screen, click `Register Service` then select `Portal`.  *Note* this doesn't create a portal yet - it just registers the service.  We'll add the portal after we create the provider org.
31. Set the Title to whatever you like then change the `Management Endpoint` to `https://pd.10.0.0.5.nip.io`.  
32. Change the `Portal Website URL` `https://pw.10.0.0.5.nip.io`.  Click Save.
33. You can then associate the analytics service to the gateway service by clicking on the `Associate Analytics` link.
34. Setup notifications for your APIC Instance.  Go to `Settings` -> `Notifications`
35. Under Sender & Email Server click `Edit`.
36. Enter in a `Name` and `Email` for your Admin user, this can be anything.
37. Down Below, click on the link that says `Configure Email Server`
38. Set up an email server of your choice, Gmail works fine here, just set it up for port 587 pointing to smtp.gmail.com.  Enter in the credentials for a gmail account.  Its not necessary to go to gmail to get the notification we will need, as once the mail is sent, we will have access to the link we need in API Connect.  Click `Save`
39. From the top left menu - go to `Provider Organizations`.
40. Click `Add` -> `Invite Organization Owner`.  Enter in a email address, use something different here.
41. Once you see the activation link, click on it.  Copy the link and paste it to a new browser tab window on the developer machine.
42. No need to fill out any of the information that pops up, API Connect will grab your credentials from ICP, so going forward the API Manager UI will use SSO facilitated via ICP.
43. You will automatically be logged into the API Manager.  From here you can continue with the next section.



Part Two: Deploy some Integration Assets
-----------------------------

1.  One your developer machine.  Go to a command prompt and execute a `git clone https://github.com/ibm-cloudintegration/techguides`.  
2.  Load up the management page for your ACE install.  You can access it from the Platform Navigator by clicking on your `ace` install or just direct your browser to `https://ace.10.0.0.5.nip.io/ace-ace-cip/`
3. Click on the `+Server` button.  On the local file system, navigate to the `/home/student/techguides` directory and find the lone bar file which should be `RESTRequest_Everything.bar`.  A pop up with a content URL will come up.  Copy this to your clipboard.  You will need it for the next step.
4. You will now deploy your bar file using the ACE chart, so you will be taken to the chart configuration screen in ICP.  Click the `Configure` button or the `Configuration` tab at the top to continue
5. Set the `Helm Release` to `lab`
6. Change the `Target Namespace` to `ace`
7. Tick the `License` checkbox
8. Click on the `All Parameters` twisty to open it up
7. Paste in the URL given to you above `Content Server URL`
8. Untick the `Production Usage` checkbox
9. Set the `Image Pull Secret` to `acekey`
10. Untick the `Enable Persistence` and `Use Dynamic Provisioning` checkboxes.
11. Under the `Service` settings, find the `NodePort IP` change this to `lab.10.0.0.5.nip.io`.  This will be the endpoint for your service running on ACE.
12. Leave the remaining settings as defaults and then click `Install` at the bottom.  Your chart will now install.  You can view the progress of the install via Helm Releases as prompted or use `kubectl`.
13. Return back to your ACE Management UI. (Click the `done` on the window from the previous step).  You should now see one service running on your integration server.  Click on the 3 dot hamburger button on the `lab` icon.  Click Open.
14. You will now see 3 icons, one for the REST Request API, Application and Shared Library.  Click on the hamburger icon for REST Request_API and then click open
15. You will then see two URL's that look something (but not exactly) like this:  

	- REST API Base URL `http://lab.10.0.0.5.nip.io:30098/restrequest_api/v1`
	- OpenAPI document `http://lab.10.0.0.5.nip.io:30098/restrequest_api/v1/Tutorial_swagger.json`

Copy and paste the link for the OpenAPI document into a browser window and then save the json file on the local file system.  We will use this inside of API Connect

16. Open up your API Manager Window inside the developer machine - `https://mgmt.10.0.0.5.nip.io/manager`.  The login should happen automatically as it will use your ICP credentials for login.
2. Click on the `Manage Catalogs`
3. Click on `Sandbox`
4. Click on `Settings` from left menu
5. Click on `Gateway Services`
6. Click `edit` and choose `Gateway`.
17. From the API Manager homescreen, use the menu on the left to navigate to `Develop`.
18. On the APIs and Products screen click `add` -> `Api`
19. Select the `From Existing API Service` radio button option.  Click `Next`.
20. Click the `browse` button to navigate to the `Tutorial_swagger.json` file on your file system you had saved previously.
21. Click `Next`
22. Review the settings then click `Next` twice (do not activate the API yet).
23. Review the summary to ensure the API was created properly, and then click the `Edit API` button.
24. Click the `Assemble` tab to bring up the Assembly editor.
25. Click the Play button to start the Test tool.
25. Click the blue `Activate API` button to deploy.
26. Select the `get /UserNumber` operation.
27. Scroll down to the `Parameters` section and find the `UserNumber` - replace what is there with the number `1`.
26. Scroll down and click the `Invoke` button.  The first time you invoke this, it will bring up a CORS warning.  Copy and paste the link for the API into a new browser tab.  It will return a 401 error for authorization, but this will load up the cert.
27. Run the invoke again, and you should see the output properly.

### Conclusion
You have completed the initial CIP Labs by taking a base ICP install and then imported some basic integration assets and exposed that as an API running on the configuration.  The key value is having the single platform that runs all of the capability that can be managed easily, user the power of Kubernetes.  

 
**End of Lab**
