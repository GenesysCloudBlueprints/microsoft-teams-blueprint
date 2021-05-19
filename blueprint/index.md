---
title: Set up Microsoft Teams meeting on Genesys Cloud
author: {author name}
indextype: blueprint
icon: blueprint
image: images/6COpenScriptDropdown.png
category: 6
summary: |
  This Genesys Cloud Developer Blueprint explains how to set up Genesys Cloud and Microsoft 365 for agents to schedule a meeting with customers using Microsoft Teams. The agents can now directly schedule a Microsoft Teams meeting from Genesys Cloud. Genesys Cloud automatically sends an SMS message with the meeting URL to the customer and also opens the Microsoft Teams for the agent. The call is inbound or outbound as long it is in a queue. 
---

This Genesys Cloud Developer Blueprint explains how to set up Genesys Cloud and Microsoft 365 for agents to schedule a meeting with customers using Microsoft Teams. The agents can now directly schedule a Microsoft Teams meeting from Genesys Cloud. Genesys Cloud automatically sends an SMS message with the meeting URL to the customer and also opens the Microsoft Teams for the agent. The call is inbound or outbound as long it is in a queue. 
The meeting scheduling solution from an agent’s point of view is shown in the illustration.

![Microsoft Teams Agent View](images/bpTeamsAgent.png)

To enable an agent to use Microsoft Teams meeting from Genesys Cloud, the operation spans multiple platforms at the system level. The public APIs offered by both Genesys Cloud and Microsoft Graph provide this interoperability. The API interaction between Genesys Cloud and Microsoft 365 is shown in the illustration.

![Microsoft Teams Integration](images/bpTeamsSystem.png)

## Solution components
* **Genesys Cloud** - A suite of Genesys cloud services for enterprise-grade communications, collaboration, and contact center management. Contact center agents use the Genesys Cloud user interface.
* **Genesys Cloud API** - A set of RESTful APIs that enables you to extend and customize your Genesys Cloud environment. The Genesys Cloud API for agentless SMS notification sends the meeting information to the caller.
* **Amazon Web Services** - Amazon Web Services (AWS), a cloud computing platform that provides a variety of cloud services such as computing power, database storage, and content delivery. AWS hosts Genesys Cloud.
* **Microsoft 365** - An integrated, cloud suite of productivity apps. This solution uses one of the apps, namely Microsoft Teams to schedule meetings.
* **Microsoft Teams** - A meeting and communication app. Microsoft Teams is the app that hosts the meeting for our solution.
* **Microsoft Graph** - A unified API endpoint that enables developers to integrate services and devices with Microsoft products. This solution uses the API endpoint to create a meeting in Microsoft Teams.
* **Microsoft Azure** - A cloud computing platform that provides a variety of cloud services for building, testing, deploying, and managing applications through Microsoft-managed data centers. Microsoft Azure hosts Microsoft 365.

## Prerequisites

### Specialized knowledge

* Administrator-level knowledge of Genesys Cloud
* Administrator-level knowledge of Microsoft Azure Active Directory
* REST API authentication
* Genesys Cloud scripting

### Genesys Cloud account

* A Genesys Cloud 3 license with agentless SMS functionality. For more information, see [Genesys Cloud Pricing](https://www.genesys.com/pricing "Opens the pricing article").
* The Master Admin role in Genesys Cloud. For more information, see [Roles and permissions overview](https://help.mypurecloud.com/?p=24360 "Opens the Roles and permissions overview article") in the Genesys Cloud Resource Center.

### Microsoft account

* Admininstrator-level role for Microsoft Azure Active Directory to set up authorization and grant permissions for Genesys Cloud. 
* Microsoft Teams license for agents.

## Implementation steps

- [Solution components](#solution-components)
- [Prerequisites](#prerequisites)
  - [Specialized knowledge](#specialized-knowledge)
  - [Genesys Cloud account](#genesys-cloud-account)
  - [Microsoft account](#microsoft-account)
- [Implementation steps](#implementation-steps)
  - [Configure Microsoft Azure custom app](#configure-microsoft-azure-custom-app)
  - [Configure Genesys Cloud](#configure-genesys-cloud)
  - [Create a Web Services Data Actions integration](#create-a-web-services-data-actions-integration)
  - [Create an OAuth Client for use with the Genesys Cloud Data Action integration](#create-an-oauth-client-for-use-with-the-genesys-cloud-data-action-integration)
  - [Create a Genesys Cloud Data Actions integration](#create-a-genesys-cloud-data-actions-integration)
  - [Load Supporting Data Actions](#load-supporting-data-actions)
  - [Import Create Teams Video Meeting data action](#import-create-teams-video-meeting-data-action)
  - [Send SMS data action](#send-sms-data-action)
  - [Load Script](#load-script)
- [Test the deployment](#test-the-deployment)
- [Additional resources](#additional-resources)

### Configure Microsoft Azure custom app
To enable the Genesys Cloud instance to authenticate and retrieve user information from the Graph API for Microsoft Teams, register your custom application in Azure 

1. Log in to the Azure portal.
2. Navigate to **App registrations**, and click **New registration**.

   ![New Registration for an Azure app](images/0ACreateNewAzureAppRegistration.png)

3. Enter a name for the application and select the preferred supported account type. Any account type works for a single Genesys Cloud organization. Click **Register**.

   ![Register the new Azure app](images/0BRegisterNewApp.png)

4. On the application's Overview page, copy **Application (client) ID** and **Directory (tenant) ID** for later use.
   
   ![Newly registered Azure app](images/0CNewApp.png)

5. Navigate to **Certificates & secrets** and click **New client secret**.
   
   ![Create client secret](images/0DCreateClientSecret.png)

6. Copy the client secret value for later use.

   **Note:** You cannot view or retrieve the client secret value later.

   ![Copy and save client secret](images/0ECopyClientSecret.png)

7. Confirm that your Genesys Cloud permission is created in Azure.
   
   ![Microsoft Azure Permissions](images/azurePerm.png)

### Configure Genesys Cloud

### Create a Web Services Data Actions integration
To enable communication from Genesys Cloud to Microsoft Azure and Microsoft Teams, you must create a Web Services Data Action integration:

1. Install the **Web Services Data Actions** integration from Genesys Cloud or from the AppFoundry. See [About the data actions integrations](https://help.mypurecloud.com/?p=209478 "Opens the Data Actions overview article").

   ![Web Services Data Actions in Genesys Cloud Integrations Search](images/1AWebServicesDataActionInstall.png)

2. Rename the Web Services Data Action and provide a short description.

   ![Rename and describe the Web Services Data Action](images/1BRenameWebServicesDataAction.png)

3. Navigate to **Configuration** > **Credentials** and click **Configure**.

   ![Navigate to Configuration -> Credentials -> Configure](images/1CConfigurationCredentials.png)

4. Create the following five fields, enter the required values, and then click **OK**:

   * A: Set the client_id to the Application (client) ID from your Azure app.
   * B: Set the client_secret to the Client Secret from your Azure app.
   * C: Set the tenant_id to the Directory (tenant) ID from your Azure app.
   * D: Set the scope to https://graph.microsoft.com/.default.
   * E: Set the grant_type to client_credentials.

   ![Create fields and values](images/1DFieldsandValues.png)

5. Import authentication data action

   The authentication data action is imported into another data action. When a new Web Services Data Actions integration is created within an org, a Custom Auth data action is also created.
   Navigate to **Integrations** > **Actions** and open the Custom Auth data action.

   ![Custom Auth Data Action associated with Web Services Data Actions Integration](images/1ECustomAuthDataAction.png)

6. Download the auth data action file *Office-365-Auth.customAuth.json* from the [microsoft-teams-blueprint](https://github.com/MyPureCloud/microsoft-teams-blueprint "Opens the GitHub repo") GitHub repository. Save this file to your local desktop to import it into Genesys Cloud.
7. Click **Import** and browse to select the downloaded file.

   ![Open Custom Authentication Data Action](images/1FOpenCustomAuthDataAction.png)

8. Click **Import Action** to import the supplied auth data action.
   
   ![Import Custom Authentication Data Action](images/1GImportCustomAuthDataAction.png)

9.  Return to the Web Services Data Action and set it to Active.

### Create an OAuth Client for use with the Genesys Cloud Data Action integration
To enable Genesys Cloud Data Action make requests to an org, you must configure authentication with Genesys Cloud using an OAuth client.

1. Navigate to **Integrations** > **OAuth** and click **Add Client**.

   ![Add an OAuth Client](images/2AAddOAuthClient.png)

2. Enter a name for the OAuth Client and select **Client Credentials** as the Grant Type. Click the **Roles** tab and assign the roles for the OAuth Client.

   **Note:** Select a custom role that includes the permission Messaging > Sms > Send. No default role includes this permission. To create a custom role, see the Custom roles information in [Roles and permissions overview](https://help.mypurecloud.com/?p=24360 "Opens the Roles and Permission overview article").

   ![Setup OAuth Client](images/2BOAuthClientSetup.png)

3. After the roles are assigned, click **Save**. Record the Client ID and Client Secret values for later use. 

   ![OAuth Client Credentials](images/2COAuthClientCredentials.png)

### Create a Genesys Cloud Data Actions integration
The Microsoft Teams video session URL is sent as an SMS to the customer from Genesys Cloud. To enable this SMS capability, you must create a Genesys Cloud Data Action integration.

1. Install the **Genesys Cloud Data Actions** integration from Genesys Cloud or the AppFoundry. For more information, see [About the data actions integrations](https://help.mypurecloud.com/?p=209478 "Opens the Data Actions overview article").

   ![Genesys Cloud Data Actions in Genesys Cloud Integrations Search](images/3AGenesysCloudDataActionInstall.png)

2. Enter a name for the Genesys Cloud Data Action.

   ![Rename Data Action and select Configuration](images/3BRenameDataAction.png)

3. In the **Configuration** tab, select **Credentials** and click **Configure**.

   ![Add OAuth Credentials](images/3CAddOAuthCredentials.png)

4. Enter the OAuth **Client ID** and **Client Secret** that you noted in the [OAuth client creation](#create-an-oauth-client-for-use-with-the-genesys-cloud-data-action-integration"). Click **OK** and save the data action.

   ![Add OAuth Credentials](images/3DOAuthClientIDandSecret.png)

5. Navigate to the main Integrations page and confirm that the SMS data action integration is set to **Active**.

   ![Integrations set to Active](images/3ESetToActive.png)

### Load Supporting Data Actions
To enable the **Send SMS** button which sends the Microsoft Teams video session URL to the customer, you must import two more data actions.
* [Import Create Teams Video Meeting Data Action](#import-create-teams-video-meeting-data-action)
* [Send SMS Data Action](#send-sms-data-action)

### Import Create Teams Video Meeting data action
The Create Teams Video Meeting data action uses the authenticated token supplied by other data actions to request a new Microsoft Teams video meeting URL.

1. Download the *Create-Teams-Meeting.custom.json* file from the [microsoft-teams-blueprint repo](https://github.com/MyPureCloud/microsoft-teams-blueprint "Opens the GitHub repo") GitHub repository. Save this file in your local desktop to import it into Genesys Cloud.

2. Navigate to **Integrations** > **Actions** and click **Import**.

   ![Import Data Action](images/4AImportDataActions.png)

3. Select the *Create-Teams-Meeting.custom.json* file and associate with the [Web Services Data Action](#create-a-web-services-data-actions-integration) that you have created and click **Import Action**.

   ![Import Create Teams Video Meeting Data Action](images/4BImportCreateTeamsVideoMeetingDataAction.png)

### Send SMS data action
This data action creates and sends an SMS message that contains the Microsoft Teams video meeting URL to the customer. The Create Teams Video Meeting data action that you configured creates the URL.

1. Download the *Send-SMS.custom.json* file from the [microsoft-teams-blueprint repo](https://github.com/MyPureCloud/microsoft-teams-blueprint "Opens the GitHub repo") GitHub repository. Save this file in your local desktop to import it into Genesys Cloud.
2. Navigate to **Integrations** > **Actions** and click **Import**.

   ![Import Data Action](images/4AImportDataActions.png)

3. Select the *Send-SMS.custom.json* file and associate with the [Web Services Data Action](#create-a-web-services-data-actions-integration) that you have created and click **Import Action**.

   ![Import Send SMS Data Action](images/5BImportSendSMSDataAction.png)

### Load Script
You need to import the script *Send-SMS-with-Teams-Video-URL.script* that references the created data actions. The script generates the **Escalate to Teams** button for agents during an active interaction with the customer. It also sends an SMS that contains the Microsoft Teams video URL to the customer.

1. Download the *Send-SMS-with-Teams-Video-URL.script* file from the [microsoft-teams-blueprint repo](https://github.com/MyPureCloud/microsoft-teams-blueprint "Opens the GitHub repo") GitHub repository. Save this file to your local desktop to import it into Genesys Cloud.
   

2. Navigate to **Admin** > **Contact Center** > **Scripts** and click **Import**.

   ![Import Script](images/6AImportScript.png)

3. Select the downloaded *Send-SMS-with-Teams-Video-URL.script* file.

   ![Import Send SMS Script](images/6BImportSendSMSScript.png)

4. To publish the script for use in an outbound message, open the imported script.

   ![Open Script Dropdown](images/6COpenScriptDropdown.png)

5. In the Script menu, click **Publish**.

   ![Publish Script](images/6DPublishScript.png)

## Test the deployment
You can test the Create Teams Video Meeting URL data action within the data action.

1. Navigate to **Admin** > **Integrations** > **Actions** and select the Create Teams Video Meeting data action.


2. Navigate to **Setup** > **Test**, enter your user, startTime, endTime and timeZone, and then click **Run Action**.

    **Note:** Enter the time in ISO-8601 format.

   ![Test Create Teams Video Meeting URL Data Action](images/7TestCreateTeamsVideoMeetingURLDataAction.png)


## Additional resources

- [Create onlineMeeting](https://docs.microsoft.com/en-us/graph/api/application-post-onlinemeetings?view=graph-rest-1.0&tabs=javascript "Opens the Microsoft graph documentation") in the Microsoft Graph API Reference
- [About Scripting](https://help.mypurecloud.com/?p=54284 "Opens the Scripting overview article") in the Genesys Cloud Resource Center
- [Agentless SMS Notifications](https://developer.mypurecloud.com/api/tutorials/agentless-sms-notifications/index.html?language=java&step=1 "Opens the SMS tutorial") in the Genesys Cloud Developer Center
- [Auto Send SMS](https://developer.mypurecloud.com/api/tutorials/sms-sending/index.html?language=nodejs&step=1 "Opens the SMS Sending tutorial") in the Genesys Cloud Developer Center
