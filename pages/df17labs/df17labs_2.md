---
title: Lab 2 (TBD)
toc: false
sidebar: df17labs
permalink: df17labs_2.html
summary: Lab 2 (TBD)
---

# Integration Scenario 2 – Create an API using flows

## Overview
Kirk is responsible for keeping track of Paradise Audiofiles dealer network and registered customer base in their CRM system, sitting within a team of business analysts. Since the merger with Big Blue Tweeters they have reconciled their dealer information into Salesforce. He is working on a project with a business partner to build a mobile app for their dealers to register service requests, and they need an API. Kirk has done some coding before, but he isn’t a developer.

 	You should have a separate print-out with credentials for:
IBM App Connect and Salesforce. Wherever you see ibmappconnect0XX then use the ID you were given.
Firefox toolbar bookmarks exist to help with navigating to those applications.

3.2	Log into IBM App Connect
__1. 	In your browser, go to https://designer.appconnect.ibmcloud.com/ and sign in with the ibmappconnect0XX@gmail.com account you have been given.
 

 	If you are not presented with a login screen, then close all browser windows and start again to make sure you’re using the right login.

3.3	Connect IBM App Connect to Salesforce
As Kirk, you know you want to build his API to create and retrieve “Case” objects in Paradise Audiophiles Salesforce CMR - because that’s what is used to track service tickets today. So you look and see if App Connect can connect to your Salesforce system.
__1. 	Using IBM App Connect in your browser, if there are any flows on the dashboard written by a previous attendee, then you may stop and delete them if you wish.
__2. 	Open the Applications tab and start typing “salesforce” …
 
__3. 	If your App Connect account is not already connected to Salesforce you will see this indicated and you should click the Connect to Salesforce button:
 
__4. 	Click continue at this dialog:
 
__5. 	You will be taken through some dialogs to authenticate and grant access to App Connect:
     
 	You might need to log into https://gmail.com with the supplied details to obtain a verification code during the sign-in process

3.4	Define your API
__1. 	Return to the dashboard and click New->Flow for an API to create an API:
 
__2. 	We want an API for managing Service Tickets. Type “Service Tickets API” as the title of the API as shown below, and name the model for the resource “ServiceTickets” being careful to ensure that it is capitalised ash shown and there is no space between ‘Service’ and ‘Tickets’. 
 
Then click Create model:
__3. 	Now we need to create some properties on our ServiceTickets model. We know we are going to be updating a “Case” in Salesforce, so we will copy some of the properties of a ‘Case’ in salesforce into our model. Click on ‘Select properties from applications’:
 
__4. 	Expand Salesforce:
 
__5. 	Then scroll down to ‘Case and expand it to see the properties of a ‘Case’ in salesforce:
 
 
__6. 	Choose the “Id”, “SuppliedEMail” and “Description” properties and click “Add Properties”. This will copy the selected properties into the Model for your API. You will be returned back to this screen with the properties you selected now filled in:
 
__7. 	Now we add “MPN” as an additional field into the model, because we always use that terminology for the part number with our dealers (regardless of how it’s referred to in our CRM), and so that’s what the mobile developers will expect:
 
 	Take care not to use spaces, to leave the “Required” checkbox unchecked on all fields, and mark the “ID” checkbox next to the field called “Id”

3.5	Implement the Create Operation
We now need to build the logic that connects this model to Salesforce.
__1. 	Click on the Operations tab, and click the drop-down to add the “Create ServiceTickets” operation:
 
__2. 	Click the button to start implementing the flow:
 
__3. 	This will open the flow editor, and show you what a sample request will look like to your new API when it’s made by an app:
 

3.6	Retrieve the contact from Salesforce
The mobile app is going to supply us just and e-mail address, but we need the full customer details to be able to create the ticket. So the first thing our flow needs to do is to retrieve the Contact from Salesforce.
__1. 	Click the “+” button to add an action to the flow:
 
__2. 	Scroll down to Salesforce and further scroll down and select the “Retrieve Contact” action:
 
 
__3. 	Choose “EMail” as the left side of the lookup condition, and click the icon on the right to display a list of references that are available in the context at this point in the flow:
 
 	We apologize if you see a lot of entries in the drop-down menu, including non-English language items and if these are not sorted alphabetically. This is a known issue at the time of writing this Lab, and will be addressed shortly.
__4. 	Choose “SuppliedEmail” from the inbound request:
 
__5. 	The configure the flow to only process the first matching Contact:
 
__6. 	You now have a fully configured retrieve operation that will look for exactly one Contact in Salesforce that matches that e-mail address that will be included in the Request parameters when the API is invoked. If zero or more-than-one are found, then the flow will report an error when it runs.
 

3.7	Lookup the product by MPN
We also want the find the full details of the product from the MPN field that was passed in on the API call.
__1. 	Click on the ‘+’ to the right of the salesforce Retrieve Contact node and add a Salesforce “Retrieve Product” operation to your flow. Then configure it as below:
 
__2. 	Now configure the flow to only process the first matching Product:
 

3.8	Create the Case in Salesforce
Now we want to create the actual Case in Salesforce, using all the details we have retrieved.
__1. 	Click on the ‘+’ to the right of the Retrieve Product node to add another node and  add a “Create Case” Salesforce action to the flow after Retrieve Product:
 
__2. 	Configure the following fields in the Case, where “Web” is typed in as a literal value, and the other fields are filled in from references in the context. Note that if you begin typing then matching values will be displayed for you to select from.
•	Some of the fields come from the incoming API request – “SuppliedEMail”, “MPN” and “Description”
•	Others come from the Salesforce Contact you retrieved – “Contact ID” and “Company”.
•	In “Description” you combine the “Product Name” you looked up in the Salesforce “Product” with the incoming “Description” from the API request.
You will notice some field mappings have a warning associated with them, like:
 

These are arrays. Click on the “How can I…?” link and see the suggestions:

For this lab example solution, indicate that you will single the first instance:
 

The mapped fields should be:

 
 
 
 
 

3.9	Return the ID of the new Service Ticket
We want the API to return an identifier that the app can use as a reference to retrieve this ticket later. We will use the Salesforce Case ID directly for this.

 	The best ways to manage IDs across systems is something we are actively exploring. If you are interested in engaging with us on this topic, we would love you sign up to our sponsored user program.
__1. 	Click on the Response action in the flow, and fill in the reply mapping for “Id” to “Case ID” from the Salesforce Create Case action (there is no need to fill in the other fields). Note that the simplest way to do this is to type ‘case’ and select Case Id as shown below.
 
__2. 	Click on the source field to set the mapping:
 

3.10	Switch on your API
Now we have the first operation on our API – to create ServiceTickets by retrieving Contacts and Products from Salesforce, and create Cases. So let’s give it a try.
__1. 	Click “Done” to exit the flow editor:
 
__2. 	Click on the menu in the top-right and select “Start API”:
 

 	If the “Start API” option is greyed out, click back into Operations -> Edit Flow to re-enter the flow editor. Then click on each Action in turn, letting the page load and then check there are no missing details. Then click “Done” and try to start the API again. There is a known issue at the time of writing. This issue is to be addressed shortly.

__3. 	Return to the dashboard; the flow API is running:
 

3.11	Create a Contact and Product in Salesforce to test
So that we can try out our API, we will create some objects in Salesforce. 

 	Please create unique objects for yourself in Salesforce, and try to make the e-mail address and Product Code in particular different to any existing objects in Salesforce. This will help avoid unexpected results due to your API finding the objects created by other users of the Lab. 
__1. 	Log into Salesforce at https://login.salesforce.com
__2. 	Click to create a new Contact:
 
__3. 	Make sure to fill in the “Name”, “Last Name” and “Email” with some suitably unique values:
 
__4. 	Open the App Launcher:
 
__5. 	Select “Products”
 
__6. 	Create a new Product with a suitably unique “Name“, and “Product Code”:
 

3.12	Test your new API operation
Now we want to test our API and see what it can do.
__1. 	Using your Chrome browser, start the Postman plugin tool using the App Launcher:
__2. 	Before we can test our API, do that we need to know the details of how our API has been exposed by App Connect. From the dashboard, open the API:
 
__3. 	Then click Manage:
 
__4. 	This shows the URL and credentials for the API. Leave the browser open as you will use the Copy to clipboard link in a minute:
 
__5. 	Go to the Postman tool launched from the Chrome browser. Click the Authorization tab, and then:
__a. 	Set the verb to POST
__b. 	Set the URL to the API of the URL. To this you must append the resource name of ServiceTickets.
__c. 	Set the authorization type to Basic Auth, and then set  the credentials of user and password
 
__6. 	 Still from within Postman, then click the Body tab, and then:
__a. 	Click raw
__b. 	Set type as JSON
__c. 	Enter the JSON request in the body pane.
__d. 	Press Send.
  
__7. 	You should see a successful HTTP response code of 201 and a JSON response returned.

3.13	Examine the new Case in Salesforce
If you navigate back to https://login.salesforce.com in your browser, and look at Cases. If they are not on the header bar, can find them via the “App Launcher” you used to find “Products” in Salesforce. 

 

 	You have now created and tested an API exposed by IBM App Connect!
You can stop here, go to another exercise, or continue and add more operations and logic to this API at your leisure.
 
4.	Thank You!

In this Lab you’ve seen how IBM App Connect lets you build flows driven by events and flows that implement an API. You have seen how the APIs are defined so that they conform to the conventions modern developers expect, and that you can build flows quickly via rich graphical tooling with multiple operations and logic.

 

Want to learn more?
If you didn’t complete the lab exercises or want to go further, then why not sign up for a free trial? Go to https://designer.appconnect.ibmcloud.com/newAccount and complete the registration.

Do you want to keep up to date or influence the product?
IBM App Connect is a rapidly evolving service in IBM Cloud. If you have been interested by what you’ve explored in the product today, then please sign up to our sponsored user program. The program provides the opportunity for you to influence how we extend and enhance the product to meet the needs of our customers.
  
Appendix A: Credentials Required

The Labs in this document use the following SaaS systems:
•	IBM App Connect
•	Gmail
•	Salesforce
•	Mailchimp

In order to complete the Lab, you will need logins to each of these services. If you are running through this Lab in an IBM hosted location, you might have been given a softcopy or hardcopy printout with a set of credentials. If so, please use them.

Otherwise, you will need to sign up to each of the systems using the instructions below. You can sign up for all of the systems up-front, or sign-up as to each system as you run through the Lab. You might want to sign up to Gmail first and then you can use that e-mail address to sign up to the other accounts.

 	Everywhere you see ibmappconnect0XX in the Lab guide, swap in the ID you have obtained for that system.

 
Gmail
Sign up at:  https://accounts.google.com/SignUp?service=mail

IBM App Connect
There are two ways to sign up to IBM App Connect

1.	Sign up to IBM App Connect outside of Bluemix here:
https://designer.appconnect.ibmcloud.com/newAccount 

2.	Create a service instance inside of IBM Bluemix:
http://ibm.biz/appconnect-bluemix 
If you do not already have an IBM Bluemix account, you will be prompted to sign up for a trial.

Salesforce
Sign up for a Developer account here: https://developer.salesforce.com/signup 

 	You must sign up for a developer account, not a trial account. A Salesforce trial account does not provide the API access required by IBM App Connect.

Mailchimp
Sign up here: https://mailchimp.com/signup/

