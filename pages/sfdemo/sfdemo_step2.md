---
title: 2. Running the demo
toc: true
sidebar: sfdemo
permalink: /sfdemo_step2.html
summary: In this step, you will run the demo!
---

## Objectives
1. Understand the story & background for the demo.
2. Understand the basic steps to trigger the integration flow.
3. Understand what to show.

## 2.1 Background
The main idea of this demonstration is to show how an end user could trigger backend processes through their use of Salesforce.com.

In preparation to visit a client, a seller will check an account status and discover a delayed shipment.

The seller will request a shipment upgrade, which the system processes automatically.

## 2.1 Trigger the flow
1. Before running the demo, you must choose which account you'll work with.  Choose any account, it doesn't really matter.

2. After choosing an account, you should manually create a Shipment record with a status set to `Delayed`.  You can then show this during the demo as the reason the seller wants to request an upgrade.

3. Once that's set up, to trigger the flow, merely edit the account and check the Request Shipment Upgrade checkbox.

4. If everything processes correctly, within about 5 seconds (the current polling interval), the checkbox should be unchecked and a new shipment record should be inserted. You will need to refresh the browser to see these changes.

## 2.2 Show the flow
1. Typically, you would walk through the previous section as the end user to explain the experience.

2. Next, you will show some plumbing. To explain the PushTopic, have the Developer Console -> Execute Anonymous Window open, and you can show the code inserted to create that trigger via the Streaming API.

3. Next, you can switch to MQ Explorer and show the Subscriptions (one is an MQ subscription that moves the message to an unused IIB.QUEUE, the other is the JMS app that moves the message to the ACP.QUEUE).

4. Staying in MQ Explorer, you can show the Queues, and since the IIB.QUEUE is not used, you can Browse data and show Properties to show the message that came out.

5. Next, move to App Connect Studio and show the MQ orchestration used to pick up the message, parse the JSON, call the Salesforce update and insert, parse and return the results.

6. I recommend taking a few minutes to recreate the orchestration from scratch to show how quickly and easily it can be built.

7. Finally, launch the App Connect WMC in Chrome and navigate to the completed orchestration.  Full logging is enabled, so you can show how every step in the orchestration provides a good amount of detail on inputs and outputs.
