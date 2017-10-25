---
title: Lab 1 (TBD)
toc: false
sidebar: df17labs
permalink: df17labs_1.html
summary: Lab 1 (TBD)
---

# Integration Scenario 1 – Connect your applications

## Overview
Cassie is a rising star in the marketing department. She is always looking for new digital tools to engage with customers, and has recently started using a SaaS service called Mailchimp to revolutionise email marketing within Paradise Audiophiles. However, she is frustrated with how much time she spends sifting through responses, and reconciling them with their Salesforce CRM system.

{% include note.html content="If doing this exercise at an event, you will probably have been given a separate hardcopy with your own lab account credentials for **IBM App Connect**, **Salesforce** and **Mailchimp**. Firefox toolbar bookmarks exist to help with navigating to those applications.    
Everywhere you see *ibmappconnect0XX* in the lab instructions, replace XX with the ID you have been assigned." %}

 
## Log into IBM App Connect
1. In your browser, go to <https://designer.appconnect.ibmcloud.com/> and sign in with the ibmappconnect0XX@gmail.com IBMid account you have been given.
 

## Create or update your mailing list in Mailchimp
Let us set up a new e-mail marketing campaign in Mailchimp.
1. Log into https://www.mailchimp.com with the credentials supplied:
   
1. Click on Lists:   
![](./images/df17labs/df1-image1.png)
 
1. If there is a list called “Audiophiles”, then click it and select all existing subscribers and delete them:
 
__4. 	If the list does not exist, create it with following information:
List Name: Audiophiles
Default From email address: ibmappconnect0XX@gmail.com
Default From name: App Connect
Reminder: You subscribed as an Audiophile using the online form

After you have saved the list, then go Settings -> List fields and …, and then add fields and configure as shown below:

 

2.4	Create a Flow in IBM App Connect
__1. 	Return to IBM App Connect in your browser and if there are any flows on the dashboard written by a previous attendee, then you may stop and delete them if you wish.
__2. 	Then, from the dashboard, click New to create a new event-driven flow:
 
__3. 	You should see the following screen where you need to choose and configure the event that should trigger this flow:
 
__4. 	Select Mailchimp as the application and “New Subscriber” as the event:
 
__5. 	If prompted, click “Connect to Mailchimp” and authorize IBM App Connect using the credentials supplied:
 
 
__6. 	Select the “Audiophiles” list:
  

2.5	Configure the Flow to create Salesforce Leads
You now need to modify the flow to add each subscriber as a sales Lead in Salesforce.
__1. 	Click the plus to add an action to the flow, and select Salesforce as the application, and “Create Lead” as the action:
¬  
__2. 	Scroll down the list of Salesforce entities to find the Leads, then click Create Lead:
 
__3. 	If prompted click “Connect to Salesforce” and log into salesforce using the details supplied:
     
 	You might need to log into https://gmail.com with the supplied details to obtain a verification code during the sign-in process
__4. 	Configure the “Create Lead” action with fields from your list in Mailchimp. To do this, start typing, or click the fields symbol to display the available fields:
 
 
 
 

2.6	Switch on your completed Flow

Flows need to be explicitly “switched on” to start them. From the flow designer, type in a meaningful name for the flow press the “Exit and switch on” button:
 
You will see that a tile appears on the dashboard indicating the flow is now running:
 
You are now ready to test your flow.

2.7	Let’s try it out!
If we generate the event in the source application we should be able to observe the effects in the target application:
__1. 	Generate the subscription by acting like a potential prospect who is signing up online. To do this, we need to get the URL for the signup subscription form. So, either go back to mailchimp in your browser or log back into https://www.mailchimp.com
__2. 	Select “Lists”, then “Audiophiles”, then “Signup forms”, then “General forms”. You should see a form as below, with a “Signup form URL”:
 
__3. 	Open the the “Signup form URL” in a new browser window, and fill in some details including the ibmappconnect0XX@gmail.com e-mail address you have been using as a login during the lab:
 
__4. 	Once you click “Subscribe to list” you will need to do a Captcha verification:
 
__5. 	Then you need to log into https://gmail.com as your ibmappconnect0XX user and click the confirmation e-mail:
 
__6. 	We want to see the result in Salesforce. Open a browser tab and log into Salesforce at https://login.salesforce.com
__7. 	Click on the Leads tab and find the new lead:

 

Open the Lead and check that the fields are set as expected.

If your Salesforce dashboard does not look like the above, then go to the App Launcher in the top left and then navigate to the Leads:

 
__8. 	END OF THIS SCENARIO


 
