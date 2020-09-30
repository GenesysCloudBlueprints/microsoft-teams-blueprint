
#  Microsoft Teams meeting on Genesys Cloud Blueprint
This Genesys Blueprint is a guide to setting up Genesys Cloud and Microsoft 365 to provide a meeting scheduling solution for agents using Genesys Cloud. Combining the two systems allows an agent to click a button in Genesys Cloud and schedule a Microsoft Teams online meeting with their caller. The agent does not need to leave Genesys Cloud to schedule the meeting with the Microsoft application. Genesys Cloud automatically sends an SMS message with the meeting URL to the customer's phone, and Genesys Cloud also opens the Microsoft Teams meeting for the agent. The call can be inbound or outbound, as long as it is in a queue. The figure below illustrates the agent's view of the meeting scheduling solution.

![Microsoft Teams Agent View](/images/bpTeamsAgent.png)


## Architecture
Although scheduling a Microsoft Teams meeting from Genesys Cloud requires only a click from the agent, the operation spans multiple platforms at the system level. The public APIs offered by both Genesys Cloud and Microsoft Graph provide this interoperability. The figure below illustrates the API interaction between Genesys Cloud and Microsoft 365.

![Microsoft Teams Integration](/images/bpTeamsSystem.png)

### Solution components:
* **Genesys Cloud** - The Genesys cloud-based contact center platform. The contact center agent uses the Genesys Cloud user interface.
* **Genesys Cloud API** - The secured and versioned public REST API that both Genesys developers and customers use. The Genesys Cloud API for agentless SMS sends the meeting information to the caller.
* **AWS** - Amazon Web Services (AWS) is Amazon's cloud platform. AWS hosts Genesys Cloud.
* **Microsoft 365** - (Formerly Office 365) Microsoft's integrated cloud platform that delivers apps, including Microsoft Teams, Word, Excel, and PowerPoint
* **Microsoft Teams** - Microsoft's meeting and communication app. Microsoft Teams is the app that hosts the meeting for our solution.
* **Microsoft Graph** - The Microsoft Graph API offers an endpoint to Microsoft 365 services. Genesys Cloud makes the POST request to the Microsoft Graph URL to set up the meeting.
* **Microsoft Azure** - Microsoft Azure is Microsoft's cloud platform. Microsoft Azure hosts Microsoft 365.

# Plan the solution
This solution requires permissions and configuration with both Genesys Cloud and Microsoft. It also requires working with public APIs and a Genesys Cloud script.
## Specialized knowledge
Implementing this solution requires experience in several areas or a willingness to learn:
* Administrator level knowledge of Genesys Cloud
* Administrator level knowledge of Microsoft Azure Active Directory
* REST API authentication
* Genesys Cloud scripting

## Genesys Cloud account required

This solution requires the SMS functionality included with a Genesys Cloud 3 license. For more information on licensing, see [Genesys Cloud Pricing](https://www.genesys.com/pricing "Opens the pricing article").

A recommended Genesys Cloud role for the solutions engineer is Master Admin or Script Designer. These roles include the required permission: Scripter > All > All. For more information on Genesys Cloud roles and permissions, see the [Roles and permissions overview](https://help.mypurecloud.com/?p=24360 "Opens the Roles and permissions overview article").

## Microsoft account required
The solution requires administrator level credentials for Microsoft Azure Active Directory that allow you to set up authorization and grant permissions for Genesys Cloud. Agents require the licensed use of Microsoft Teams.
# Deployment steps

## Configure Microsoft Azure Custom App
You must register your custom application within Azure to enable the Genesys Cloud instance to authenticate and retrieve user information from the Graph API for Teams.

1. Log in to Azure, navigate to **App Registrations**, and click **New Registration**.

![New Registration for an Azure app](/images/0ACreateNewAzureAppRegistration.png)

2. Name the new application and select the preferred supported account type. Any account type works for a single Genesys Cloud organization.

![Register the new Azure app](/images/0BRegisterNewApp.png)

3. View the new application. Copy the Application and Directory IDs for later use.

![Newly registered Azure app](/images/0CNewApp.png)

4. Navigate to **Certificates & Secrets** and click **+ New Client Secret**.

![Create client secret](/images/0DCreateClientSecret.png)

5. Copy the client secret value for later use.
  **Warning:** You cannot view or retrieve the client secret value later.

![Copy and save client secret](/images/0ECopyClientSecret.png)
6. Confirm that your Genesys Cloud permission is created in Azure.
![Microsoft Azure Permissions](/images/azurePerm.png)

## Configure Genesys Cloud
### Create a Web Services Data Actions Integration
To enable communication from Genesys Cloud to Microsoft Azure and Microsoft Teams, you need to create a Web Services Data Action integration and provide it with the proper fields and values:

1. Search for and install the **Web Services Data Actions** integration from within Genesys Cloud or from the AppFoundry. For more information, see [About the data actions integrations](https://help.mypurecloud.com/?p=209478 "Opens the Data Actions overview article").

![Web Services Data Actions in Genesys Cloud Integrations Search](/images/1AWebServicesDataActionInstall.png)

2. Rename the Web Services Data Action with your desired name and provide a short description.

![Rename and describe the Web Services Data Action](/images/1BRenameWebServicesDataAction.png)

3. Navigate to **Configuration > Credentials** and click **Configure**.

![Navigate to Configuration -> Credentials -> Configure](/images/1CConfigurationCredentials.png)

4. Create the following five fields, provide the desired values, and then click **OK**:

- A: Set the client_id to the Application (client) ID from your Azure app.
- B: Set the client_secret to the Client Secret from your Azure app.
- C: Set the tenant_id to the Directory (tenant) ID from your Azure app.
- D: Set the scope to https://graph.microsoft.com/.default.
- E: Set the grant_type to client_credentials.

![Create fields and values](/images/1DFieldsandValues.png)

5. Import Authentication Data Action

The Authentication data action is imported into another data action.  When a new Web Services Data Actions Integration is created within an org, a Custom Auth data action is also created.

![Custom Auth Data Action associated with Web Services Data Actions Integration](/images/1ECustomAuthDataAction.png)

6. Open this Custom Auth data action.
7. The *Office-365-Auth.customAuth.json* file provided in the [microsoft-teams-blueprint repo](https://github.com/MyPureCloud/microsoft-teams-blueprint "Opens the GitHub repo") on GitHub is the supplied Auth Data Action. Download this file and save it to your desktop for import into Genesys Cloud.
8. Click **Import** and **Import Action** to import the supplied Auth Data Action.

![Open Custom Authentication Data Action](/images/1FOpenCustomAuthDataAction.png)

![Import Custom Authentication Data Action](/images/1GImportCustomAuthDataAction.png)

9. Return to the Web Services Data Action and confirm it is set to Active.

### Create an OAuth Client for use with the Genesys Cloud Data Action Integration
To enable a Genesys Cloud Data Action to make requests to an org, you must configure authentication with Genesys Cloud using an OAuth client.

1. Navigate to **Integrations > OAuth** and click **+Add Client**.

![Add an OAuth Client](/images/2AAddOAuthClient.png)

2. Enter a name for the new OAuth Client and select the Grant Type **Client Credentials**. Click the **Roles** tab and select roles for the OAuth Client.

**Note:** You must select a custom role that includes the permission Messaging > SMS > Send. No default role includes this permission. To create a custom role, see the Custom roles information in the [Roles and permissions overview](https://help.mypurecloud.com/?p=24360 "Opens the Roles and Permission overview article").


![Setup OAuth Client](/images/2BOAuthClientSetup.png)


3. Once role(s) are selected, click the **Client Details** tab of the OAuth Client. Record the Client ID and Client Secret values for later use.

![OAuth Client Credentials](/images/2COAuthClientCredentials.png)

### Create a Genesys Cloud Data Actions Integration
This solution uses SMS within Genesys Cloud to send the Microsoft Teams video session URL to the customer. To enable this SMS capability, you need to create a Genesys Cloud Data Action Integration.

1. Search for and install the **Genesys Cloud Data Actions** integration from within Genesys Cloud or from the AppFoundry. For more information, see [About the data actions integrations](https://help.mypurecloud.com/?p=209478 "Opens the Data Actions overview article").

![Genesys Cloud Data Actions in Genesys Cloud Integrations Search](/images/3AGenesysCloudDataActionInstall.png)

2. Enter a desired name for the Genesys Cloud Data Action. Click the **Configuration** tab and then the **Credentials** tab.

![Rename Data Action and select Configuration](/images/3BRenameDataAction.png)

3. Click **Configure** and enter the OAuth **Client ID** and **Client Secret** you created previously. Click **OK**.

![Add OAuth Credentials](/images/3CAddOAuthCredentials.png)

![Add OAuth Credentials](/images/3DOAuthClientIDandSecret.png)

4. Click **Save**. Navigate to the main Integrations page and confirm the SMS Data Actions integration is set to **Active**.

![Integrations set to Active](/images/3ESetToActive.png)

### Load Supporting Data Actions
To enable the **Send SMS** button containing a MS Teams video session link to a customer, you need to import two additional data actions.

#### Import Create Teams Video Meeting Data Action
The Create Teams Video Meeting data action uses the authenticated token supplied by other data actions to request a new Microsoft Teams video meeting URL.

1. The *Create-Teams-Meeting.custom.json* file provided in the [microsoft-teams-blueprint repo](https://github.com/MyPureCloud/microsoft-teams-blueprint "Opens the GitHub repo") on GitHub. Download this file and save it to your desktop for import into Genesys Cloud.

2. Navigate to **Integrations > Actions** and click **Import**.

![Import Data Action](/images/4AImportDataActions.png)

3. Locate the *Create-Teams-Meeting.custom.json* file. Associate it with the Web Services Data Action you created previously.
4. Click **Import Action**.

![Import Create Teams Video Meeting Data Action](/images/4BImportCreateTeamsVideoMeetingDataAction.png)

#### Send SMS Data Action
This data action creates and sends to the customer an SMS message containing the Microsoft Teams video meeting URL. The URL is created by the Create Teams Video Meeting Data Action you configured.

1. The *Send-SMS.custom.json* file provided in the [microsoft-teams-blueprint repo](https://github.com/MyPureCloud/microsoft-teams-blueprint "Opens the GitHub repo") on GitHub is the Send SMS Data Action. Download this file and save it to your desktop for import into Genesys Cloud.
2. Navigate to **Integrations > Actions** and click **Import**.

![Import Data Action](/images/4AImportDataActions.png)

3. Locate the *Send-SMS.custom.json* file you downloaded and associate it with the Web Services Data Action created previously.
4. Click **Import Action**.

![Import Send SMS Data Action](/images/5BImportSendSMSDataAction.png)

### Load Script
You need to load a script that references the data actions. This script generates the Escalate to Teams button for agents during an active interaction. It also sends the SMS to the customer containing the Teams video URL.

1. The *Send-SMS-with-Teams-Video-URL.script* file is provided in the [microsoft-teams-blueprint repo](https://github.com/MyPureCloud/microsoft-teams-blueprint "Opens the GitHub repo"). Download this file and save it to your desktop for import into Genesys Cloud.

2. Navigate to **Admin > Contact Center > Scripts** and click **Import**.

![Import Script](/images/6AImportScript.png)

2. Select the *Send-SMS-with-Teams-Video-URL.script* file you downloaded.

![Import Send SMS Script](/images/6BImportSendSMSScript.png)

3. To publish the script for use in an outbound message, open the imported script. Click the Script menu.

![Open Script Dropdown](/images/6COpenScriptDropdown.png)

4. Click **Publish**.

![Publish Script](/images/6DPublishScript.png)

## Test the deployment
You can test the Create Teams Video Meeting URL data action within the data action itself.

1. Navigate to **Admin > Integrations > Actions** and select the Create Teams Video Meeting data action.

2. Navigate to **Setup > Test**, insert your user, startTime, endTime, and timeZone, then click **Run Action**.
**Note:** Enter times in ISO-8601 format.

![Test Create Teams Video Meeting URL Data Action](/images/7TestCreateTeamsVideoMeetingURLDataAction.png)


## Additional resources

 -  [Create onlineMeeting section](https://docs.microsoft.com/en-us/graph/api/application-post-onlinemeetings?view=graph-rest-1.0&tabs=javascript "Opens the Microsoft graph documentation") in the Microsoft Graph API Reference
- [About Scripting](https://help.mypurecloud.com/?p=54284 "Opens the Scripting overview article") in the Genesys Cloud Resource Center
- [Agentless SMS Notifications Tutorial](https://developer.mypurecloud.com/api/tutorials/agentless-sms-notifications/index.html?language=java&step=1 "Opens the SMS tutorial") in the Genesys Cloud Developer Center
- [Auto Send SMS Tutorial](https://developer.mypurecloud.com/api/tutorials/sms-sending/index.html?language=nodejs&step=1 "Opens the SMS Sending tutorial") in the Genesys Cloud Developer Center
