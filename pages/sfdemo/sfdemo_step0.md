---
title: Step 0 - Setup Salesforce.com
toc: true
sidebar: sfdemo
permalink: /sfdemo_step0.html
summary: In this step, you will establish your Salesforce.com developer account and create the custom fields and objects required by the demo.
---

## Objectives
1. Establish a Developer Userid/Password on Salesforce.com.
2. Customize the Salesforce Objects to match the demo requirements.
3. Capture the security token required by the demo components.

## 0.1 Create Developer Account

##### Setting up an account

1. Go to [developer.salesforce.com](http://developer.salesforce.com) and click **Sign up**. ![Sign up](/images/sfdemo_step0_signup.png)

2. Complete and submit the registration form.  
   **Note**: You will need a valid and accessible email account to complete the registration.

3. Login to your email account and locate the new message from Salesforce.com with the subject **Salesforce.com login confirmation**.

4. Follow the link to complete the registration and setup a new password for your account.

5. That's it!

## 0.2 Customize Salesforce Objects

1. Login to [developer.salesforce.com](http://developer.salesforce.com).

2. Click your name in the top-right corner of the screen, and select **My Developer Account**.

3. If prompted, log in again.

4. Click the gear icon in the top-right corner of the screen, and select **Setup**.

#### Add the `Request Shipment Upgrade` field to the Account Object

5. On the left navigation, under **Objects and Fields**, select **Object Manager**.

6. Click on **Account**.

7. Scroll down, under **Fields & Relationships**, click **New**.

8. Select **Checkbox**, and click **Next**.

9. For **Field Label**, enter `Request Shipment Upgrade`.

10. For **Default Value**, leave it **Unchecked**.

11. For **Field Name**, enter `RequestShipmentUpgrade` (i.e., remove the generated underscores).

12. Click **Next** for Steps 2 and 3, and click **Save** on Step 4.

#### Create the `Shipment` Object

1. On the left navigation, under **Objects and Fields**, select **Object Manager**.

2. Click **Create** -> **Custom Object**.

3. For **Label**, enter `Shipment`.

4. For **Plural Label**, enter `Shipments`.

5. For **Object Name**, enter `Shipment`.

6. Click **Save**. 

7. Similar to steps 3 - 8 above, create the following fields:

| Data Type | Related To | Field Label | Length | Values | Field Name | Required |
| --- | --- | --- | --- | --- | --- | --- |
| Lookup Relationship | `Account` | `Account` | _n/a_ | _n/a_ | `Account` | Yes |
| Text | _n/a_ | `Description` | 200 | n/a | `Description` | Yes |
| Picklist | _n/a_ | `Priority` | _n/a_ | `Standard, Expedited` | `Priority` | Yes |
| Picklist | _n/a_ | `Status` | _n/a_ | `Started, Shipped, Delivered, Delayed` | `Status` | Yes |

#### Create the PushTopic

1. Click the gear icon in the top-right corner of the screen, and select **Developer Console**.

2. Click **Debug** -> **Open Execute Anonymous Window**.

3. Paste the following code and click **Execute**.

```
PushTopic pushTopic = new PushTopic();
pushTopic.Name = 'ShipmentUpgradeRequests';
pushTopic.Query = 'SELECT Id, AccountNumber, Name FROM Account WHERE RequestShipmentUpgrade__c = true';
pushTopic.ApiVersion = 39.0;
pushTopic.NotifyForOperationCreate = true;
pushTopic.NotifyForOperationUpdate = true;
pushTopic.NotifyForFields = 'Where';
insert pushTopic;
```

## 0.3 Reset Security Token

1. Login to [developer.salesforce.com](http://developer.salesforce.com).

2. Click your name in the top-right corner of the screen, and select **My Developer Account**.

3. If prompted, log in again.

4. Click your avatar icon in the top-right corner of the screen, and select **Settings**.

5. On the left navigation, under **My Personal Information**, select **Reset My Security Token**.

6. Click the button to **Reset Security Token**.

7. Login to your email account and locate the new message from Salesforce.com with the subject **Your new Salesforce security token**.

## Conclusion

You are now ready to begin the demo!