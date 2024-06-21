---
title: Trunk Selection Prefix
author: barrez.bert
indextype: blueprint
icon: blueprint
image: images/
category: 4
summary: |
---
Genesys Cloud Trunk Selection Prefix
---
This Genesys Cloud Developer Blueprint explains how Genesys Cloud trunk selection is handled by configuration on Sites. Number Plans and Outbound Routes ultimately determine the target trunk by matching the dialed number against a prioritized Number Plan, classifying it, and attempting the highest priority trunk matching the classification. This is a blueprint is a solution for idea INB-I-1146 (https://genesyscloud.ideas.aha.io/ideas/INB-I-1146).

Telephony Administrators can steer different types of calls through different trunks, assuming there is something unique to the dialed number scheme that can be matched against using one of the supported match types. Match types include Digit Length, E.164 Number List, Inter-Country, Intra-Country, Number List, or RegEx.

## Solution components

* **Genesys Cloud** - A suite of Genesys cloud services for enterprise-grade communications, collaboration, and contact center management. Contact center agents use the Genesys Cloud user interface.
* **Genesys Cloud API** - A set of RESTful APIs that enables you to extend and customize your Genesys Cloud environment.
* **Trunks** - A telecommunications circuit from a carrier to a Genesys Cloud telephony connection. 
* **Sites** - the home of a set of phones. The site defines the call classification rules and outbound routing rules as well as the telephony properties for dialing.


## Prerequisites

### Specialized knowledge

* Administrator-level knowledge of Genesys Cloud
* Telephony Configurations
* Have an established set Trunk to configure Sites

### Genesys Cloud account

* A Genesys Cloud CX 1 license. For more information, see [Genesys Cloud Pricing](https://www.genesys.com/pricing "Opens the Genesys Cloud pricing article").
* The Master Admin role in Genesys Cloud. For more information, see [Roles and permissions overview](https://help.mypurecloud.com/?p=24360 "Opens the Roles and permissions overview article") in the Genesys Cloud Resource Center.

## Configure Genesys Cloud

### Assign permissions to access Telephony Configurations

1. Navigate to **Admin** > **Roles/Permissions**.
2. Ensure you have **Telephony**>**All Permissions** permission.
3. Click **Save** to assign the appropriate permissions to your role.

   ![Telephony Permissions](images/permissions.png "Telephony Permissions")

## Sites 

### Create a Site

Trunks are put on routes within sites.

1. Go to **Admin**>**Telephony**>**Sites**
2. Create a site. For example purposes, I will create a site called “Trunk Selection Prefix”. 
2. You can create special Number Plans.
3. Save Number Plan
4. You can specify Outbound Routes
5. Save Outbound Routes


### Create a Number Plan
1. Click "New Number Plan" to create a new Number Plan
2. Input Number Plan Name
3. Input Match Type
4. Input Match Exptression
5. Input Normalized Number Expression
6. Input Classification 

#### Here are two examples:

Example 1: Forced *88 capture and handling (International)
1. Created a new Number Plan called "Forced *88 capture and handling (International)" 
2. Match Type set to "Regular Expression"
3. Match Exptression set to "^(\*88)(.*)$"
5. Normalized Number Expression set to "$1$2"
6. Classification customized to "Forced Trunk Prefix *88". NOTE: You can input custom text and press ENTER if you want to custom classify. 
7. Save Number Plan

   ![Forced *88 capture and handling](images/forced88.png "Forced *88 capture and handling")

Example 2: Forced *99 capture and handling (Domestic)
1. Created a new Number Plan called "Forced *99 capture and handling (Domestic)" 
2. Match Type set to "Regular Expression"
3. Match Exptression set to "^(\*99)(.*)$"
5. Normalized Number Expression set to "$1$2"
6. Classification customized to "Forced Trunk Prefix *99". NOTE: You can input custom text and press ENTER if you want to custom classify. 
7. Save Number Plan

   ![Forced *99 capture and handling](images/forced99.png "Forced *99 capture and handling")


## Data Action

You will need to create a Genesys Cloud data action that will be used for disconnecting interactions. This can be called “Disconnect interaction”. 

### Create an OAuth client for use with a Genesys Cloud data action integration

To enable a Genesys Cloud data action to make public API requests on behalf of your Genesys Cloud organization, use an OAuth client to configure authentication with Genesys Cloud.

Create an OAuth client to use with the data action integration with a custom role.

To create an OAuth Client in Genesys Cloud:

1. Navigate to **Admin** > **Integrations** > **OAuth** and click **Add Client**.

2. Enter the name (Example: Disconnect Interaction) for the OAuth client and select **Client Credentials** as the grant type. Click the **Roles** tab and assign the required role for the OAuth client.

3. Click **Save**. Copy the client ID and the client secret values for later use.

   **Note:** Ensure that you **copy the client ID and client secret values** for each of the OAuth clients.

### Add Genesys Cloud data action integration

Add a Genesys cloud data action integration for each OAuth client being used with this blueprint to call the Genesys Cloud public API to:
* Terminate the call of an inbound blacklisted caller

To create a data action integration in Genesys Cloud:

1. Navigate to **Admin** > **Integrations** > **Integrations** and install the **Genesys Cloud Data Actions** integration. For more information, see [About the data actions integrations](https://help.mypurecloud.com/?p=209478 "Opens the About the data actions integrations article") in the Genesys Cloud Resource Center.

2. Enter a name for the Genesys Cloud data action, such as "Disconnect Interaction" in this blueprint solution.

3. On the **Configuration** tab, click **Credentials** and then click **Configure**.

4. Enter the client ID and client secret that you saved for the Public API (OAuth Client 1). Click **OK** and save the data action.

5. Navigate to the Integrations page and set the data action integration to **Active**.

   ![create data action](images/create-data-action.gif "create data action")


### Import the Genesys Cloud data actions

1. Download the `Disconnect-Interaction.json` JSON file from the [ani-blacklist](https://github.com/GenesysCloudBlueprints/ani-blacklist) GitHub repository.
2. In Genesys Cloud, navigate to **Admin** > **Integrations** > **Actions** and click **Import**.
3. Select the `Disconnect-Interaction.json` file and associate with "Disconnect Interaction" data action integration, which uses the Disconnect Interaction Public API OAuth client.
4. click **Import Action**.
5. Click **Save & Publish**

   ![create data action](images/import-data-actions.gif "create data action")

## Architect 

### Import the Architect workflows

This solution includes one Architect workflow that uses one [data action](#add-genesys-cloud-data-action-integrations "Goes to the Add a web services data actions integration section"). 

* The **Blacklist.i3WorkFlow** workflow is triggered when a blacklisted caller dials to Genesys Cloud communicate user. This workflow terminates an inbound phone call if it matches the phone number from the Data Table. 

First import this workflow to your Genesys Cloud organization:

1. Download the `Blacklist.i3WorkFlow` file from the [ani-blacklist repo](https://github.com/GenesysCloudBlueprints/ani-blacklist) GitHub repository.

2. In Genesys Cloud, navigate to **Admin** > **Architect** > **Flows:Workflow** and click **Add**.

3. Enter a name for the workflow and click **Create Flow**.

4. From the **Save** menu, click **Import**.

5. Select the downloaded **Blacklist.i3WorkFlow** file and click **Import**.

6. Review your workflow. Click **Save** and then click **Publish**.
   
   ![Import the workflow](images/architect-workflows.gif "Import the workflow")

   **Note:** If you imported the `Blacklist.i3WorkFlow` file, your workflow will look like the flow below. 

   ![full architect workflow](images/full-architect-workflow.gif "full architect workflow")

## Triggers

Create the trigger that invokes the created Architect workflow.

1. From Admin Home, search for **Triggers** and navigate to the Triggers list.

2. From the Triggers list, click **Add Trigger**

3. From the Add New Trigger modal, name your trigger and click **Add**

4. From the Trigger single view input **Topic Name**, **Workflow Target**, and **Data Format** as mentioned in the table below.  

| Topic Name | Workflow Target | Data Format |
|---------------------------------------------------|-----------|--------------------|
| v2.detail.events.conversation.{id}.customer.start | Blacklist | TopLevelPrimitives |

5. Click **Add Condition**.  
   NOTE: For more information, see [Available Topics](https://developer.genesys.cloud/notificationsalerts/notifications/available-topics "Opens the Available Topics article") in the Genesys Cloud Developer Center. Using the notification monitoring tool in the Developer Center, you can watch the notifications happen.

6. From the Trigger single view, input **JSON Path**, **Operator**, and **Value** as mentioned in the table below. 

| Topic Name | Workflow Target | Data Format |
|------------|-----------------|-------------|
| mediaType | Equals(==) | VOICE |

7. Click **Save**.

   ![Configure Trigger](images/trigger.gif "Configure Trigger")

### You can now add numbers to the Blacklist Data Table under the ani column header and those callers will be disconnected when they try calling you.  

## Additional resources

* [Genesys Cloud API Explorer](https://developer.genesys.cloud/devapps/api-explorer "Opens the GC API Explorer") in the Genesys Cloud Developer Center
* [Genesys Cloud notification triggers](https://developer.genesys.cloud/notificationsalerts/notifications/available-topics "Opens the Available topics page") in the Genesys Cloud Developer Center
* The [ani-blacklist](https://github.com/GenesysCloudBlueprints/ani-blacklist) repository in GitHub
