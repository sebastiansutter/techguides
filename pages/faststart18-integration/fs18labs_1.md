---
title: Lab 1
toc: false
sidebar: fs18labs
permalink: fs18labs_1.html
summary: Simple guided application-to-application integration
---
 
# Integration Scenario 1 – Connect your applications

## Overview
![](./images/df17labs/cassie.png)  
Cassie has just been assigned to a new project team where she has taken on a new role as an integration developer. The project has very recently gone live with SAP Hybris, and the decision was made by the project lead to use Salesforce.com as the CRM System of Record, and to make sure that Hybris and Salesforce are keeping information in sync.  

The project requirements are broken down into 2 phases.  Phase 1 is the migration of existing contacts from Salesforce to Hybris.  Phase 2 is the ongoing synchronization of new and updated contacts into Hybris.  For this lab you will be completing phase 1 and part of phase 2.

{% include note.html content="Refer to the [Logon Credentials](fs18labs_creds.html) page for the userids and passwords you should use on your machine." %}

Note also you will need your own Salesforce.com account.  You can get one for free from www.sforce.com.  This will be your own permanent developer account.  Note - Make sure you have created a developer account and not a trial account.  Trial accounts to do not have access to the Saleforce API, and thus will not work with AppConnect.

 
## Log into IBM App Connect and create the OAuth Token Generator API for SAP Hybris
#### The SAP Hybris system you are using has its own API secured using OAuth.  You will need to have your integration get an Oauth token before making any subsequent API Calls.  The steps below will guide you through the process of creating an API that will allow you do this inline with your flow

1. Launch the Chrome browser. 
2. log into the IBM Cloud (aka Bluemix) [https://console.bluemix.net/]()
3. If you haven't done so already, Add the App Connect service to your list of services, under Integration.  
4. Launch the App Connect service
1. Create a new API Flow `+New -> Flows for an API`
2. Create a new Model.  call that model `token`
3. Create four properties for the model: `userid`, `password`,  `bearertoken`, and `expiresin`.
4. Click on `Operations`
5. Click on `Select Operation to Add`
6. Select `Create Token`.  This will create the HTTP Post entry to be able to create the Oauth bearer token
7. Click on `Implement Flow`.  This opens the Flow Designer
4. Add in a `HTTPInvoke` operation to the flow.
5. Configure the `HTTP Invoke` by following these steps:
	* Set the method to `POST`
	* Set the URL to:  `http://cap-sg-prd-4.integration.ibmcloud.com:18447/authorizationserver/oauth/token?client_id=resttest&client_secret=WaBbvHBxECw6ZGkz&grant_type=password&username=keenreviewer11@hybris.com&password=2LFPz45d3QskX65Y`
	* Set the headers to: `{"Accept":"application/json","Content-Type":"application/json"}`
8. Add a `JSONParse` operation after the `HTTPInvoke`
9. The `JSONInput` to use will be the `Response Body` found in the `HTTP/Invoke Method`.  Click on the triple bar Icon to the right of the text field to select the proper input variable.
9. Use the following JSON sample for the response:
>
>{  
	>"access_token": "...",  
	>"token_type": "bearer",  
	>"refresh_token": "...",  
	>"expires_in": 3599,  
	>"scope": "basic"  
>}  

10. Click on 	`Generate Schema` to populate the JSON Schema section of the operation

11. Back to the main flow, click the `Response` operation to populate the JSON response. Set the following variables.  You can copy and paste the following, or use the browse capability to select the proper variable
	* userid -> `{{$Request.userid}}`
	* bearertoken -> `{{$JSONParserParse.access_token}}`
	* expiresin -> `{{$JSONParserParse.expires_in}}`
12. You can now start up the Flow by clicking the `Done` button then the icon in the upper right hand corner that looks like 3 dots.  Click that and then select `Start API` you will see that the status will change to `Running`.
13. Click on the Manage section of the new flow you created.  Here you will see the information about how to use the IBM Cloud Native API Management capability.
14. Scroll down to `Sharing Outside of Bluemix Organization` and click on `Create API Key`.  
15. Copy the `API KEY` and then click on the `API PORTAL LINK` link. You can explore your API in the portal and invoke it by clicking on `Try it`.
16. To call the API, you will need to paste in your `X-IBM-Client-ID` and then on `Call Operation` if it is successful, you will see the JSON output with your token id.
17. Be sure to note the following information about your service, as you will be calling it in other portions of this lab.

* Specific items to note and keep handy for later are:
	* Route to invoke your API (you can find this in the manage section) You will also 	need to append the model name to the end of the URL - e.g. 	`https://	service.us.apiconnect.ibmcloud.com/gws/apigateway/api/	1234567889009293939399/abcDEF0/model` 
	* Model name in this case is, 	`token`.  Note your URL will be different.
	* The API Key, which can be found at the bottom section of the manage screen under the `Sharing Outside of Cloud Foundry organization` heading. 

## Configure the Flow to do the initial load of SAP Hybris Contacts as Salesforce Contacts

2. Create a new API Flow from the main landing page in Designer and go to `+New -> Flows for an API`
2. For this flow, create a model called `contactaddress`
3. Create one new property.  Call it `numberofcontactssynced`
4. Click on `Operations` and then `Implement Flow`
5. Add a `HTTP Invoke` to your flow for the call to get the Oauth Token
6. Configure the `HTTP Invoke` with the following values:
	* 

## Create the Synchronization Flow that will sync Contacts from Saleforce into Addresses in SAP Hybris on an ongoing basis
Let us set up a new Flow that will take contacts we have in Salesforce, and take the new Contacts that are created, and have them automatically create a new Address in SAP Hybris for that Contact

1. Make sure you are still logged into App Connect
2. Create a link to your Salesforce Account by going to `Applications` and add your Salesforce instance.  App Connect will step you through the process of granting App Connect Access to your Salesforce instance.
3. CLick the `+NEW` icon and create a new Event Driven Flow
4. From the `How do you want to start the flow` list, you can browse down to `Salesforce` or use the search box to limit list.
5. 

 
You are now ready to test your flow.

## Let’s try it out!


## Thank You!

In this Lab you’ve seen how IBM App Connect provides features for simple, guided app-to-app integration. With IBM App Connect you can link your applications so that when something happens in one application, other relevant applications get updated automatically, so you can:

* Stop wasting time on repetitive manual tasks
* Use this intuitive business tool to take back control
* Link your apps in a few simple steps


### Do you want to keep up to date or influence the product?
IBM App Connect is a rapidly evolving service in IBM Cloud. If you have been interested by what you’ve explored in the product today, then please sign up to our sponsored user program. The program provides the opportunity for you to influence how we extend and enhance the product to meet the needs of our customers.    
![](./images/df17labs/df2-image45.png)