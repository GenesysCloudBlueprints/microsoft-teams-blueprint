---
title: Set up Microsoft Teams meeting on Genesys Cloud
author: yuri.yeti
indextype: blueprint
icon: blueprint
image: images/6COpenScriptDropdown.png
category: 6
summary: |
  This Genesys Cloud Developer Blueprint explains how to set up Genesys Cloud and Microsoft 365 for agents to schedule a meeting with customers using Microsoft Teams. The agents can now directly schedule a Microsoft Teams meeting from Genesys Cloud. Genesys Cloud automatically sends an SMS message with the meeting URL to the customer and also opens the Microsoft Teams for the agent. The call can be either inbound or outbound as long it is in a queue.
---

This Genesys Cloud Developer Blueprint explains how to set up Genesys Cloud and Microsoft 365 for agents to schedule a meeting with customers using Microsoft Teams. The agents can now directly schedule a Microsoft Teams meeting from Genesys Cloud. Genesys Cloud automatically sends an SMS message with the meeting URL to the customer and also opens the Microsoft Teams for the agent. The call can be either inbound or outbound as long it is in a queue.
The following illustration shows the meeting scheduling solution from an agent’s point of view.

![Microsoft Teams agent view](images/msteams-workflow.png "Meeting scheduling solution to use Microsoft Teams from an agent's point of view")

The end to end customer and agent experience is shown in the following illustration.

![Overview](images/teamsvideo_gif.gif "Overview")

To enable an agent to use Microsoft Teams meeting from their Genesys Cloud agent UI, you use several public APIs that are available from Genesys Cloud and Microsoft Graph. The following illustration shows the API calls between Genesys Cloud and Microsoft 365.

![Microsoft Teams integration](images/microsoft-teams-architect.png "The API calls between Genesys Cloud and Microsoft Graph API")

## Contents
* [Solution components](#solution-components "Goes to the Solution components section")
* [Prerequisites](#prerequisites "Goes to the Prerequisites section")
* [Implementation steps](#implementation-steps "Goes to the Implementation steps section")
* [Additional resources](#additional-resources "Goes to the Additional resources section")

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

* An enterprise Azure account. Personal Azure accounts do not support the OAuth grant type of client credentials in Genesys Cloud.
* Admininstrator-level role for Microsoft Azure Active Directory to set up authorization and grant permissions for Genesys Cloud.
* Microsoft Teams license for agents.

## Implementation steps
* [Configure the Microsoft Azure custom app](#configure-the-microsoft-azure-custom-app "Goes to the Configure the Microsoft Azure custom app section")
* [Configure Genesys Cloud](#configure-genesys-cloud "Goes to the Configure Genesys Cloud section")
* [Add a web services data actions integration](#add-a-web-services-data-actions-integration "Goes to the Add a web services data actions integration section")
* [Create an OAuth client for use with the Genesys Cloud data action integration](#create-an-oauth-client-for-use-with-the-genesys-cloud-data-action-integration "Goes to the Create an OAuth client for use with the Genesys Cloud data action integration section")
* [Add a Genesys Cloud data actions integration](#add-a-genesys-cloud-data-actions-integration "Goes to the Add a Genesys Cloud data actions integration section")
* [Load the supporting data actions](#load-the-supporting-data-actions "Goes to the Load the supporting data actions section")
* [Import Create Teams video meeting data action](#import-create-teams-video-meeting-data-action "Goes to the Import Create Teams video meeting data action section")
* [Send SMS data action](#send-sms-data-action "Goes to the Send SMS data action section")
* [Import and publish the script](#import-and-publish-the-script "Goes to the Import and publish the script section")
* [Test the deployment](#test-the-deployment "Goes to the Test the deployment section")
* [Additional resources](#additional-resources "Goes to the Additional resources section")

### Configure the Microsoft Azure custom app
To enable the Genesys Cloud instance to authenticate and retrieve user information from the Graph API for Microsoft Teams, register your custom application in Azure.

1. Log in to the Azure portal.
2. Navigate to **Manage** > **App registrations** and click **New registration**.

   ![New registration for an Azure app](images/0ACreateNewAzureAppRegistration.png "Azure portal for new registration")

3. Enter a name for the application and select the preferred supported account type. Any account type works for a single Genesys Cloud organization. Click **Register**.

   ![Register the new Azure app](images/0BRegisterNewApp.png "Registration details for new application")

4. On the application's **Overview** page, copy the **Application (client) ID** and **Directory (tenant) ID** for later use.

   ![Newly registered Azure app](images/0CNewApp.png "Newly registered application in Azure portal")

5. Navigate to **Certificates & secrets** and click **New client secret**.

   ![Create client secret](images/0DCreateClientSecret.png "Registered application certificates and secrets")

6. Copy the client secret value for later use.

   :::primary
   **Note:** You can't view or retrieve the client secret value later.
   :::

   ![Copy and save the client secrets](images/0ECopyClientSecret_08_21.png "Copy and save the client secrets")

7. Navigate to **Manage** > **API Permissions**. Under Configured permissions, select **Add a permission**.

   ![Microsoft Azure permissions](images/addApiPermissions.png "Add API Permissions")

8. Select the **Microsoft APIs** tab, and then select **Microsoft Graph**.

   ![Microsoft Graph](images/selectGraphApi.png "Click Microsoft Graph")

9. Select **Application permissions**.

    ![Application Permission](images/selectApplicationPermissions.png "Click Application Permissions")

10. Under the **Select permissions** section, search for Calendars in the Search field. In the search result, expand **Calendars** and select the **Calendars.ReadWrite** permission. Click **Add permissions**.

    ![Calendar Permissions](images/selectCalendarsReadWritePermission.png "Click Application Permissions")

11. Under Configured permissions, select **Grant admin consent for MS Teams Demo**.

    ![Grant Admin Consent](images/grantAdminConsent.png "Click Grant Admin Consent")    

12. To grant consent for the requested permission, click **Yes**.

    ![Confirm Grant Admin Consent](images/confirmAdminConsent.png "Confirm Grant Admin Consent")    

13. View the admin consent granted for Microsoft Graph.

    ![Admin Consent Granted](images/adminConsentGranted.png "Admin Consent Granted")    

### Configure Genesys Cloud

### Add a web services data actions integration
To enable communication from Genesys Cloud to Microsoft Azure and Microsoft Teams, you must add a web services data actions integration:

1. Install the **Web Services Data Actions** integration from Genesys Cloud. See [About the data actions integrations](https://help.mypurecloud.com/?p=209478 "Opens the data actions overview article").

   ![Web services data actions in Genesys Cloud integrations search](images/1AWebServicesDataActionInstall.png "Install web service data actions in Genesys Cloud")

2. Rename the web services data action and provide a short description.

   ![Rename and short description for the web services data action](images/1BRenameWebServicesDataAction.png "Rename and provide short description for web services data action")

3. Navigate to **Configuration** > **Credentials** and click **Configure**.

   ![Credentials configuration](images/1CConfigurationCredentials.png "Configure credentials for web services data action")

4. For **User Defined (OAuth)** credential type, add the following five fields, enter the required values, and then click **OK**:

* A: Set the client_id to the Application (client) ID from your Azure app.
* B: Set the client_secret to the Client Secret from your Azure app.
* C: Set the tenant_id to the Directory (tenant) ID from your Azure app.
* D: Set the scope to https://graph.microsoft.com/.default.
* E: Set the grant_type to client_credentials.

   ![Configure credential fields and values](images/1DFieldsandValues.png "Select credential type and add field names and values")

5. Import authentication data action

   You import an authentication data action into another data action. When you add a new web services data actions integration within an organization, Genesys Cloud creates a **Custom Auth** data action automatically.

   Navigate to **Integrations** > **Actions** and open the **Custom Auth** data action.

   ![Custom Auth data action associated with web services data actions integration](images/1ECustomAuthDataAction.png "Open the custom auth data action associated with web services data action")

   1. At the bottom of the Custom Auth data action page, click **Viewing** to change the status of the data action from **Published** to **Draft**.
   2. Download the auth data action file `Office-365-Auth.customAuth.json` from the [microsoft-teams-blueprint](https://github.com/MyPureCloud/microsoft-teams-blueprint "Opens the GitHub repo") GitHub repository. Save this file to your local desktop to import it into Genesys Cloud.
   3. Click **Import** and browse to select the downloaded file.

      ![Import authentication data action file](images/1FOpenCustomAuthDataAction.png "Import the custom authentication data action file")

   4. Click **Import Action** to import the custom auth data action.

      ![Import custom authentication data action](images/1GImportCustomAuthDataAction.png "Import the custom auth data action for web services data action")

   5. Click **Save & Publish**.
      :::primary
      **Note:** Click **Yes** in the Publish Action window to publish the data action. The import action modifies only the data action configuration and not the data action contract.
      :::

   6.  Return to the web services data action integration and verify that the data action is in **Active** status.

### Create a custom role for use with Genesys Cloud OAuth client

1. Navigate to **Roles/Permissions** and click **Add Role**.

   ![Create Custom Role](images/createRole.png "Create Custom Role")

2. Enter a name for your custom role.

   ![Name Custom Role](images/nameCustomRole.png "Name Custom Role")

3. In the **Permissions** tab, search and select the **Conversation > message > Create** and **messaging > sms > send** permissions. Click **Save** to assign the selected permissions to your custom role.

   :::primary
   **Note:** Assign this role to your user record before creating the Genesys Cloud OAuth client. The **messaging > sms > send** permission requires the "GMA/Portico: Non conversational bi-directional SMS, MMS, Email and RCS" messaging product to be activated in your Genesys Cloud organization.
   :::

   ![Add Permissions to Custom Role](images/assignPermissionToCustomRole.png "Add Permissions to Custom Role")


### Create an OAuth client for use with the Genesys Cloud data action integration
To enable Genesys Cloud data action make requests to an organization, you must configure authentication with Genesys Cloud using an OAuth client.

1. Navigate to **Integrations** > **OAuth** and click **Add Client**.

   ![Add an OAuth client](images/2AAddOAuthClient.png "Add an OAuth client")

2. Enter a name for the OAuth client and select **Client Credentials** as the Grant Type. Click the **Roles** tab and assign the roles for the OAuth client.

   :::primary
   **Note:** Select a custom role that includes the permission **messaging > sms > send**. No default role includes this permission. To create a custom role, see the Custom roles information in [Roles and permissions overview](https://help.mypurecloud.com/?p=24360 "Opens the Roles and Permission overview article").
   :::

   ![Set up OAuth client](images/2BOAuthClientSetup.png "Select custom role and the grant type for client details")

3. Click **Save** and record the Client ID and Client Secret values for later use.

   ![OAuth client credentials](images/2COAuthClientCredentials.png "Copy the client ID and secret values of the OAuth client")

### Add a Genesys Cloud data actions integration
The Microsoft Teams video session URL is sent as an SMS to the customer from Genesys Cloud. To enable this SMS capability, you must add a Genesys Cloud data actions integration.

1. Install the **Genesys Cloud Data Actions** integration from Genesys Cloud. For more information, see [About the data actions integrations](https://help.mypurecloud.com/?p=209478 "Opens the Data Actions overview article").

   ![Genesys Cloud Data Actions in Genesys Cloud integrations search](images/3AGenesysCloudDataActionInstall.png "Install Genesys Cloud data actions")

2. Enter a name for the Genesys Cloud data action.

   ![Rename the data action](images/3BRenameDataAction.png "Rename the data action")

3. In the **Configuration** tab, select **Credentials** and click **Configure**.

   ![Add OAuth Credentials](images/3CAddOAuthCredentials.png "Configure credentials")

4. Enter the OAuth **Client ID** and **Client Secret** that you noted in the [OAuth client creation](#create-an-oauth-client-for-use-with-the-genesys-cloud-data-action-integration "Goes to the Genesys Cloud data action section"). Click **OK** and save the data action.

   ![Add OAuth client credentials](images/3DOAuthClientIDandSecret.png "Enter the OAuth client ID and secret")

5. Navigate to the main Integrations page and set the SMS data action integration to **Active**.

   ![SMS data integrations set to active](images/3ESetToActive.png "Set the SMS data action integration to active status")

### Load the supporting data actions

To enable the **Send SMS** button that sends the Microsoft Teams video session URL to the customer, you must import two more data actions.
* [Import Create Teams video meeting data action](#import-create-teams-video-meeting-data-action "Goes to the Import Create Teams video meeting data action section")
* [Send SMS data action](#send-sms-data-action "Goes to the Send SMS data action section")

### Import Create Teams video meeting data action
The Create Teams Video Meeting data action uses the authenticated token supplied by other data actions to request a new Microsoft Teams video meeting URL.

1. Download the `Create-Teams-Meeting.custom.json` file from the [microsoft-teams-blueprint repo](https://github.com/MyPureCloud/microsoft-teams-blueprint "Opens the GitHub repo") GitHub repository. Save this file in your local desktop to import it into Genesys Cloud.

2. Navigate to **Integrations** > **Actions** and click **Import**.

   ![Import the data action](images/4AImportDataActions.png "Import the data action")

3. Select the `Create-Teams-Meeting.custom.json` file and associate with the web services data action that you created in the [Add a web services data actions integration](#add-a-web-services-data-actions-integration "Goes to the Add a web services data actions integration section") section and click **Import Action**.

   ![Import the Create Teams video meeting data action](images/4BImportCreateTeamsVideoMeetingDataAction.png "Select the Create Teams meeting JSON file to import it")

### Send SMS data action
This data action creates and sends an SMS message that contains the Microsoft Teams video meeting URL to the customer. The Create Teams Video Meeting data action that you configured creates the URL.

1. Download the `Send-SMS.custom.json` file from the [microsoft-teams-blueprint repo](https://github.com/MyPureCloud/microsoft-teams-blueprint "Opens the GitHub repo") GitHub repository. Save this file in your local desktop to import it into Genesys Cloud.
2. Navigate to **Integrations** > **Actions** and click **Import**.

   ![Import the data action](images/4AImportDataActions.png "Import the data action")

3. Select the `Send-SMS.custom.json` file and associate with the web services data action you created in the [Add a web services data actions integration](#add-a-web-services-data-actions-integration "Goes to the Add a web services data actions integration section") section and click **Import Action**.

   ![Import the Send the SMS data action](images/5BImportSendSMSDataAction.png "Import the SMS data action")

### Import and publish the script
Import the `Send-SMS-with-Teams-Video-URL.script` file that references the created data actions. The script creates the **Escalate to Teams** button for agents during an active interaction with the customer. When the agent clicks the button, it sends an SMS that contains the Microsoft Teams video URL to the customer.

1. Download the `Send-SMS-with-Teams-Video-URL.script` file from the [microsoft-teams-blueprint repo](https://github.com/MyPureCloud/microsoft-teams-blueprint "Opens the GitHub repo") GitHub repository. Save this file to your local desktop to import it into Genesys Cloud.  

2. Navigate to **Admin** > **Contact Center** > **Scripts** and click **Import**.

   ![Import the script](images/6AImportScript.png "Import the script")

3. Select the downloaded `Send-SMS-with-Teams-Video-URL.script` file.

   ![Import the send SMS script](images/6BImportSendSMSScript.png "Import the SMS script")

4. To configure the script for use in an outbound message, open the imported script.

   ![Open the Script menu](images/6COpenScriptDropdown.png "Open the Script menu")

5. Click the **Actions** icon and then select **Escalate to Teams**.

   ![Click Script Actions](images/clickScriptActions.png "Click Script Actions")

6. Expand the first data action **Data Actions.Execute Data Action**, that creates the Teams meeting.

   ![Expand the First Data Action](images/expandFirstDataAction.png "Expand the First Data Action")

7. From the Category list, select the Teams meeting web services data action. From the Data Action list, select the Teams meeting data action.

   ![Expand the First Data Action](images/mapFirstDataAction.png "Expand the First Data Action")

8. Expand the input variables for the first data action and enter the user input variable. The variable can be either an agent's Teams account or the Teams account that must be used regardless of which agent is on the interaction.
   :::primary
   **Note:** When you enter an agent's Teams account, the agent's Genesys Cloud email address must match their Teams email address. When you define a general Teams account, it can be either an email address or an object ID of an agent in the Azure Activity Directory.
   ::: 

      ![Map First Data Action Input Variable](images/mapFirstDataActionDefineUserInputVariable.png "Map First Data Action Input Variable")

9. Define the static values for the input variables, `startTime`, `endTime`, and `timeZone`. 

   :::primary
   **Note:** You can enter the same value for `startTime` and `endTime` parameters and the timestamp can be in the past. Also, define your own timestamp format. The example shows the value to create the ISO-8601 format timestamp. 
   :::

   ![Define Static Input Variables](images/mapFirstDataActionDefineInputVariables.png "Define Static Input Variables")

10. Expand the second data action that sends the SMS.

    ![Expand the second data action.](images/expandSecondDataAction.png "Expand the second data action.")

11. From the Category list, select the category of the Send SMS data action.  From the Data Action list, select the Send SMS data action.

    ![Map Second Data Action](images/mapSecondDataAction.png "Map Second Data Action")

12. Expand the input variables for the second data action.

13. In the **fromAddress** input variable, type one of your SMS numbers bought from the Genesys Cloud organization.

      :::primary
      **Note:** See [Purchase SMS long code numbers](https://help.mypurecloud.com/articles/purchase-sms-long-code-numbers/ "Opens the Purchase SMS long code numbers article") in the Genesys Cloud Resource Center for detailed steps about purchasing an SMS number. You must enter the SMS number in +11234567890 format.
      :::

    ![Define From Address Input Variable](images/mapSecondDataActionFromAddressVariable.png "Define From Address Input Variable")

14. Enter the values for the remaining input variables of the second data action as shown in the example.

    ![Map Second Data Action Input Variables](images/mapSecondDataActionVariables.png "Map Second Data Action Input Variables")

15. Under the Custom Action Name section, click **Save**.

    ![Save Custom Action](images/saveScriptCustomAction.png "Save Custom Action")

16. In the Script menu, click **Save**.

    ![Save the script](images/saveScript.png "Save the script")

17. In the Script menu, click **Publish**.

    ![Publish the script](images/6DPublishScript.png "Publish the script")

18. Navigate to **Admin** > **Contact Center** > **Queues** and select the queue that you want to associate with this script.

19. In the **Voice** tab, select the `Send-SMS-with-Teams-Video-URL` script from the **Default Script** list.

    ![Select Default Script](images/selectDefaultScriptForQueue.png "Select the Default Script")

20. Click **Save & Continue**.

### Test the deployment
Test the Create Teams video meeting URL data action within the data action.

1. Navigate to **Admin** > **Integrations** > **Actions** and select the Create Teams Video Meeting data action.


2. Navigate to **Setup** > **Test**, enter the **user**, **startTime**, **endTime** and **timeZone**, and then click **Run Action**.

    :::primary
    **Note:** The **startTime** and **endTime** parameters must be in ISO-8601 format. The **user** parameter can be either the agent's Azure Active Directory Object ID or the agent's Teams email address.
    :::

   ![Test the Create Teams video meeting data action](images/7TestCreateTeamsVideoMeetingURLDataAction.png "Test the deployment")

Test the Send SMS data action within the data action.

1. Navigate to **Admin** > **Integrations** > **Actions** and select the Send SMS data action.
2. Navigate to **Setup** > **Test**, and enter the **fromAddress** and **toAddress**.

   :::primary
   **Note:** Use one of the purchased SMS numbers within your Genesys Cloud org for the **fromAddress** parameter.
   :::
3. Enter **sms** for **toAddressMessengerType** and click **Run Action**.

   ![Test the Send SMS data action](images/testSendSMSDataAction.png "Test the Send SMS deployment")

## Additional resources

- [Create onlineMeeting](https://docs.microsoft.com/en-us/graph/api/application-post-onlinemeetings?view=graph-rest-1.0&tabs=javascript "Opens the Microsoft graph documentation") in the Microsoft Graph API Reference
- [Purchase SMS long code numbers](https://help.mypurecloud.com/articles/purchase-sms-long-code-numbers/) in the Genesys Cloud Resource Center
- [About Scripting](https://help.mypurecloud.com/?p=54284 "Opens the Scripting overview article") in the Genesys Cloud Resource Center
- [Agentless SMS Notifications](https://developer.mypurecloud.com/api/tutorials/agentless-sms-notifications/index.html?language=java&step=1 "Opens the SMS tutorial") in the Genesys Cloud Developer Center
- [Auto Send SMS](https://developer.mypurecloud.com/api/tutorials/sms-sending/index.html?language=nodejs&step=1 "Opens the SMS Sending tutorial") in the Genesys Cloud Developer Center
