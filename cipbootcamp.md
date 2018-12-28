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

**Part One**:  Use of the Cloud Integration Platform
**Part Two**:  Reference Materials:  Installation of the Cloud Integration Platform


Part one contains the hands on section of the lab where you will get acquainted with the Platform Navigator and be setting up the individual components of the platform and deploying some basic assets.

Part two is an informational section that provides you key information about your environment and provides guidance should you ever have to install your own environment.  *You will not be installing CIP from scratch as part of this lab*.



Part One: Hands on acclimation exercise with the Cloud Integration Platform
-------------------------------------------

1.  You have been provided a pre-installed environment of the Cloud Integration Platform.  It includes a vanilla ICP 3.1.1 install with the CIP Platform Navigator installed.
2. The environment you are using consists of 9 different nodes.  8 of which are ICP Nodes and one is a developer image that you will be using to access the ICP User Interfaces.  You can access this VM directly using the Skytap interface.
3. If required, you are also able to SSH into your environment.  You can find this information in your Skytap window under *networking*.  You can do the majority of your CLI work from the Master node.
4. Before you can execute any of the `kubectl` commands you will need to execute a `cloudctl login`.  The credentials for the system are `admin`/`admin`.
5. Password-less SSH has also been enabled between the Master node and the other nodes in the environment.  **note** a table with the environment configuration is provided below.  Credentials for each machine are `student`/`Passw0rd`.  You are able to `su` to the root user using the same password if you need to. 

| VM        | Hostname  | IP Address | # of CPU | RAM    | Disk Space (LOCAL) | Shared Storage | Additional Notes |
|-----------|-----------|------------|----------|--------|--------------------|----------------|------------------|
| Master    | master    | 10.0.0.1   | 12       | 32 GB  | 400 GB             | N/A            |                  |
| Proxy     | proxy     | 10.0.0.5   | 4        | 8 GB   | 20 GB              | N/A            |                  |
| Worker 1  | worker-1  | 10.0.0.2   | 8        | 16 GB  | 400 GB             | 200 GB (Ceph)  | Ceph Master      |
| Worker 2  | worker-2  | 10.0.0.3   | 8        | 16 GB  | 400 GB             | 200 GB (Ceph)  | OSD 1            |
| Worker 3  | worker-3  | 10.0.0.4   | 8        | 16 GB  | 400 GB             | 200 GB (Ceph)  | OSD 2            |
| Worker 4  | worker-4  | 10.0.0.7   | 8        | 16 GB  | 400 GB             |                |                  |
| Worker 5  | worker-5  | 10.0.0.8   | 8        | 16 GB  | 400 GB             |                |                  |
| Worker 6  | worker-6  | 10.0.0.9   | 8        | 16 GB  | 400 GB             |                |                  |
| Developer | developer | 10.0.0.6   |          |        |                    |                |                  |


6. Let's start by having you access Developer Instance from the Skytap UI.  Click on the Developer Machine, and it will take you directly to the Developer Machine running X-Windows.  Should you need to Authenticate, you can use the credentials of `student`/`Passw0rd!`.
7. Bring up the Firefox browser.  Navigate to the main ICP UI by going to `https://10.0.0.1:8443`.  The credentials again are `admin/admin`.  Here you have access to all of the typical ICP functions.
8. Open up a new tab in the browser to bring up the ICIP Platform Navigator.  The ICIP Platform navigator can be found by navigating to `https://xxxxx`.
9. The Platform Navigator UI can be use to create and manage instances of all of the components that make up the Cloud Integration Platform. **note** at the time of this lab, the ability to create and manage Aspera instances has not yet be added to the Platform Navigator, and will be added at a later date.

Part Two: Installation/Configuration Reference for the Cloud Integration Platform
-------------------------------------------

1.  Although you won't be installing the entire Cloud Integration Platform as part of this lab, it is important to understand how it is installed and how to properly size an environment for Demo or POC purposes.  

2. Below you will find the structure of the CIP environment used for this bootcamp.  It represents the bare minimum to run CIP in any environment for development or POC purposes only.  As is evident below, a sizable amount of resources are required.  

| VM        | Hostname  | IP Address | # of CPU | RAM    | Disk Space (LOCAL) | Shared Storage | Additional Notes |
|-----------|-----------|------------|----------|--------|--------------------|----------------|------------------|
| Master    | master    | 10.0.0.1   | 12       | 32 GB  | 400 GB             | N/A            |                  |
| Proxy     | proxy     | 10.0.0.5   | 4        | 8 GB   | 20 GB              | N/A            |                  |
| Worker 1  | worker-1  | 10.0.0.2   | 8        | 16 GB  | 400 GB             | 200 GB (Ceph)  | Ceph Master      |
| Worker 2  | worker-2  | 10.0.0.3   | 8        | 16 GB  | 400 GB             | 200 GB (Ceph)  | OSD 1            |
| Worker 3  | worker-3  | 10.0.0.4   | 8        | 16 GB  | 400 GB             | 200 GB (Ceph)  | OSD 2            |
| Worker 4  | worker-4  | 10.0.0.7   | 8        | 16 GB  | 400 GB             |                |                  |
| Worker 5  | worker-5  | 10.0.0.8   | 8        | 16 GB  | 400 GB             |                |                  |
| Worker 6  | worker-6  | 10.0.0.9   | 8        | 16 GB  | 400 GB             |                |                  |
| Developer | developer | 10.0.0.6   |          |        |                    |                |                  |

3. This lab environment uses Skytap to managed the infrastructure on Softlayer, but it can be deployed in any public/private cloud where ICP is supported.

4. Before starting your installation, be sure to make sure all prerequisites and setup steps have been completed.  Follow these steps [here](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_2.1.0.3/installing/install_app_mod.html), take special care in ensuring you can completed the steps under *Configuring your cluster*. 

![](./images/pots/cip/skytap.png)

4. CIP is provided as a single download file that can be uploaded to what will become your master node.  

5. Shared Storage is *required* for persistence to be supported on all components.  Ceph is required for API Connect, and Gluster is required for App Connect Enterprise, MQ and Event Streams.  **Note** all components *except* for API Connect can be installed without persisence being enabled.  You *must* have Ceph shared storage set up before hand.

5. This will take you to the catalog screen. 

6. 

7. 






Building a simple integration flow
-----------------------------

1.  While logged into your App Connect Instance, in the main dashboard, Click on the **New+** button and then select **Flows for an API**.


	![](./images/pots/ace-designer/acplab1newflow.png)

1.  Enter in a name for your new flow.  Call it **FirstFlow**.  Also, APIs built in the Designer are model based.  Let's name our mode **sample**.  Click on **Create Model**.

	![](./images/pots/ace-designer/acplab1newflow2.png)

1.  You will now define your model.  The structure of the model itself will be JSON based, but with the App Connect Designer, you can define this model graphically by adding *properties*.  The Properties can either be simple objects, such as a *String*.  They can also be complex objects like an *object* or an *array*.  For this lab, define three properties for our model as defined below.  Use the **+Add Property** to add a new property after the first.  **Note** App Connect will automatically save any changes to your flow when you make them, so there is no need to manually save your work.

	* firstname - Type String
	* lastname - Type String
	* message - Type String
	
	![](./images/pots/ace-designer/acplab1newflow3.png)

2.  Next to the **Properties** heading you will see a second heading called **Operations**.  Here we will define the operations for an API.  In this case, we will be doing a create operation, select the **Create sample** option from the list.

	![](./images/pots/ace-designer/acplab1newflow4.png)

1.  You will see that the **Create Sample** Operation created a new *POST* resource `/sample`.  You will now define this API by clicking on **Implement flow**

	![](./images/pots/ace-designer/acplab1newflow5.png)

2.  You will then see the framework of what the API looks like.  There is the request structure, some space in the middle for where you will define the structure of the API, and then the response.  Note the Request body example that took the model you just created and shows in JSON format.  Click on the **+** icon on the flow to add  the first operation.

	![](./images/pots/ace-designer/acplab1newflow6.png)
	
3. The first operation we will add is a Gmail activity that will send a mail whenever we invoke our API.  You can filter the list of connnectors by typing into the search bar.  Enter in **Gmail** and the connector list will limited to the Gmail operations.  Click on **Create Email** to move to the next step.

	![](./images/pots/ace-designer/acplab1newflow7.png)
	
	
4. In order to work with your Gmail account, you need to connect to and grant access to App Connect to be able to work with your gmail account to send emails.  Click the **Connect** button to start this process.  You will be asked to log into your Gmail account.

	![](./images/pots/ace-designer/acplab1newflow8.png)
	
5. Click the **Allow** button to complete the process..

	![](./images/pots/ace-designer/acplab1newflow9.png)
	
6. Next, you can populate the email with hard coded values.  

	* To:  This can be your Gmail account or another address where you would like the email to be send
	* Subject:  use `Hello`
	* Body:  use `a quick test note`

	![](./images/pots/ace-designer/acplab1newflow10.png)

7. Back to the Flow Framework.  Click on the **Response** Icon.  Here you need to populate response.  Normally, this would be a response body that would be sent back to the requestor who invoked your API, but for this basic lab, just hard code your first name in the **firstname** field in the Response Body.  Click the **Done** button in the upper right hand corner of the screen.

	![](./images/pots/ace-designer/acplab1newflow11.png)

8. To switch on the API, click on the 3 dot icon in the upper right hand corner.  Select **Start API**.

	![](./images/pots/ace-designer/acplab1newflow12.png)
	
9. You should see the status of the API change to **Running**.  Click on the **Manage** tab to view the runtime information and how you can test the API.

	![](./images/pots/ace-designer/acplab1newflow13.png)

10. Here you will the see the API inside of the IBM Cloud Native API Management that is powered by IBM API Connect.  In order to make this  API executable, we will need to get an API Key.  Scroll down to the bottom and find the **Sharing Outside of Cloud Foundry organization** section.  Click the **Create API Key+** button.

	![](./images/pots/ace-designer/acplab1newflow13b.png)

11. When the create API Key dialogue comes uo, enter in a name for your API Key in the **Descriptive Name** field.

	![](./images/pots/ace-designer/acplab1newflow14.png)

12. Once that is completed, you should see the API Portal Link section popuated with a URL.  Click on that link to go to your API Developer portal page.

	![](./images/pots/ace-designer/acplab1newflow15.png)

13. The API Developer Page is designed to be a mechanism where you can socialize your APIs to others to be able to use or embed within their application.  There are key bits of information here a developer would be keen to find if they want to get documentation and endpoint information.  There is even a place where the API can be downloaded in OpenAPI format.  For this lab, you will use the test tool in the portal in order to execute and validate your API.  Click on the **Try It** on the right hand side to use the test tool.

	![](./images/pots/ace-designer/acplab1newflow16.png)
	
14. Click on the **Try It** on the right hand side to use the test tool.  The test tool will generate some sample data for you if you click **Generate**.  You can modify the test data if you wish.  Click **Call Operation** to invoke the API.

	![](./images/pots/ace-designer/acplab1newflow17.png)

15. Once the API runs, you can view the results at the bottom. To double check that everything worked, go into your inbox where you sent your mail to.  You should see your mail there.

	![](./images/pots/ace-designer/acplab1newflow18.png)

16. Now that we have the basics out of the way, lets augment our flow to include an additional step that willl call out to a new endpoint.  Close down the developer portal and return back to App Connect. Before you can make changes, you need to stop your API.  Click on the  three dots icon and then click **Stop API**. Then click on the **Define** heading.

	![](./images/pots/ace-designer/acplab1newflow19.png)

17. Click the **Operations** tab and then select **Edit Flow**.  **Note** if you only see a **View Flow** instead of **Edit Flow** then this means your API is still running.  This happens as you are not allowed to make changes to your flow while it is still running, although you can view it.  Follow the instructions in the previous step to do that.

	![](./images/pots/ace-designer/acplab1newflow20.png)

18. Add a new operation by clicking on the large **+** icon in the flow.  Filter the list of connectors under Applications to include the **Salesforce** connector.  Click on **Create account**.

	![](./images/pots/ace-designer/acplab1newflow21.png)

19. You will then see the new Salesforce Create Account operation in your flow.  Click the **Connect** button twice to start the process of connecting your Salesforce.com account to App Connect.

	![](./images/pots/ace-designer/acplab1newflow22.png)

20. You will see a pop up that reminds you that Salesforce.com API access is required for App Connect to work with your Salesforce instance.  Click **Continue**.

	![](./images/pots/ace-designer/acplab1newflow23.png)

21. You will then see Salesforce prompt for access by App Connect to your Salesforce account.  Click **Allow** to complete the process.

	![](./images/pots/ace-designer/acplab1newflow24.png)

22. You can then populate the mappings into your Salesforce account.  Lets use a dynamic value that is coming in from the model we defined earlier. Click on the **Account Name** field and then click the "Hamburger" Icon on the right hand side of the text box.  Here you can select variables that have been already defined and made available for use within the integration.  Click on the **firstname**  field.  Repeat the process for **lastname**.  You can add a space in between the firstname and lastname fields if you wish.

	![](./images/pots/ace-designer/acplab1newflow25.png)

23. Let's apply a function to do some transformation.  Click the **lastname** field box on the Account Name. and then select **Apply a function**.

	![](./images/pots/ace-designer/acplab1newflow26.png)

24. This will bring up a functions list.  Transformation functions in App Connect are done via JSONata.  The Functions tab here allows you to apply these functions quickly.  Find the **Uppercase** function under the String Functions heading.  Click on it.

	![](./images/pots/ace-designer/acplab1newflow27.png)

25. You will then see Account name with the applied transformation on the **lastname** field.

	![](./images/pots/ace-designer/acplab1newflow28.png)

8. Switch on the API, click on the 3 dot icon in the upper right hand corner.  Select **Start API**.

	![](./images/pots/ace-designer/acplab1newflow12.png)
	
9. Repeat the same process as above by going back into the developer portal by clicking **Manage** and then find the URL for your portal at the bottom.  Use the **Try it** option and then **Generate** new input test data.  You should see the email sent as before and additionally, a new account will be generated in Salesforce using the information provided in the input test data.

### Conclusion
You have completed the "Hello World" section of the App Connect Designer labs.  You should have a good feel for the UI now, and the subsequent labs will dive into some more complex use cases that are based on real world examples.

 
**End of Lab**
