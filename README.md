---
page_type: sample
languages:
- csharp
- powershell
products:
- azure
description: "This sample demonstrates how to use MSAL.NET to authenticate the user silently using username and password and call to a web API (in this case, the Microsoft Graph)"
urlFragment: aad-username-password-graph
---

# .NET Core Console application letting users sign-in with Username/password to call Microsoft Graph API

[![Build status](https://identitydivision.visualstudio.com/IDDP/_apis/build/status/AAD%20Samples/.NET%20client%20samples/active-directory-dotnetcore-console-up-v2-CI)](https://identitydivision.visualstudio.com/IDDP/_build/latest?definitionId=693)

## About this sample

### Overview

This sample demonstrates how to use MSAL.NET to:

- authenticate the user silently using username and password.
- and call to a web API (in this case, the [Microsoft Graph](https://graph.microsoft.com))

![Topology](./ReadmeFiles/Topology.png)

If you would like to get started immediately, skip this section and jump to *How To Run The Sample*.

### Scenario

The application obtains a token through username and password, and then calls the Microsoft Graph to get information about the signed-in user and their manager.

Note that Username/Password is needed in some cases (for instance devops scenarios) but it's not recommanded because:

- This requires having credentials in the application, which does not happen with the other flows.
- The credentials should only be used when there is a high degree of trust between the resource owner and the client and when other authorization grant types are not
   available (such as an authorization code).
- Do note that this attempts to authenticate and obtain tokens for users using this flow will often fail with applications registered with Azure AD. Some of the situations and scenarios that will cause the failure are listed below  
  - When the user needs to consent to permissions that this application is requesting. 
  - When a conditional access policy enforcing multi-factor authentication is in force.
  - Azure AD Identity Protection can block authentication attempts if this user account is compromised.
  - The user's pasword is expired and requires a reset.

while this flow seems simpler than the others, applications using these flows often encounter more problems as compared to other flows like authorization code grant. The error handling is also quiet complex (detailed in the sample)

The modern authentication protocols (SAML, WS-Fed, OAuth and OpenID), in principal, discourages apps from handling user credentials themselves. The aim is to decouple the authentication method from an app. Azure AD controls the login experience to avoid exposing secrets (like passwords) to a website or an app.

This enables IdPs like Azure AD to provide seamless single sign-on experiences, enable users to authenticate using factors other than passwords (phone, face, biometrics) and Azure AD can block or elevate authentication attempts if it discerns that the user’s account is compromised or the user is trying to access an app from an untrusted location and such.

## About the code

The code for handling the token acquisition process is simple, as it boils down to calling the `AcquireTokenByUsernamePasswordAsync` method of `PublicClientApplication` class. See the `GetTokenForWebApiUsingUsernamePasswordAsync` method in `PublicAppUsingUsernamePassword.cs`.

```CSharp
private async Task<AuthenticationResult> GetTokenForWebApiUsingUsernamePasswordAsync(IEnumerable<string> scopes, string username, SecureString password)
{
 AuthenticationResult result = null;
 try
 {
  result = await App.AcquireTokenByUsernamePasswordAsync(scopes, username, password);
 }
 catch (MsalUiRequiredException ex)
 {
   ...
   // error handling omited here (see sample for details)
 }
```

## How to run this sample

To run this sample, you'll need:

- [Visual Studio 2017](https://aka.ms/vsdownload) or just the [.NET Core SDK](https://www.microsoft.com/net/learn/get-started)
- An Internet connection
- A Windows machine (necessary if you want to run the app on Windows)
- An OS X machine (necessary if you want to run the app on Mac)
- A Linux machine (necessary if you want to run the app on Linux)
- An Azure Active Directory (Azure AD) tenant. For more information on how to get an Azure AD tenant, see [How to get an Azure AD tenant](https://azure.microsoft.com/en-us/documentation/articles/active-directory-howto-tenant/)
- A user account in your Azure AD tenant. This sample will not work with a Microsoft account (formerly Windows Live account). Therefore, if you signed in to the [Azure portal](https://portal.azure.com) with a Microsoft account and have never created a user account in your directory before, you need to do that now.

### Step 1: Clone or download this repository

From your shell or command line:

```Shell
git clone https://github.com/Azure-Samples/active-directory-dotnetcore-console-up-v2.git
```

or download and exact the repository .zip file.

> Given that the name of the sample is pretty long, and so are the name of the referenced NuGet packages, you might want to clone it in a folder close to the root of your hard drive, to avoid file size limitations on Windows.

#### Operating the sample

When you run the sample, if you are running on a domain joined or AAD joined Windows machine, it will display your information as well as the information about your manager.

### Step 2: (Optional) Register the sample with your Azure Active Directory tenant

The instructions so far used the sample is for an app in a Microsoft test tenant: given that the app is multi-tenant, anybody can run the sample against this app entry.

There is one project in this sample. To register it, you can:

- either follow the steps [Step 2: Register the sample with your Azure Active Directory tenant](#step-2-register-the-sample-with-your-azure-active-directory-tenant) and [Step 3:  Configure the sample to use your Azure AD tenant](#choose-the-azure-ad-tenant-where-you-want-to-create-your-applications)
- or use PowerShell scripts that:
  - **automatically** creates the Azure AD applications and related objects (passwords, permissions, dependencies) for you
  - modify the Visual Studio projects' configuration files.

If you want to use this automation:
1. On Windows run PowerShell and navigate to the root of the cloned directory
1. In PowerShell run:
   ```PowerShell
   Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process -Force
   ```
1. Run the script to create your Azure AD application and configure the code of the sample application accordinly. 
   ```PowerShell
   .\AppCreationScripts\Configure.ps1
   ```
   > Troubleshooting information as well as documentation about other ways of running the scripts is available in [App Creation Scripts](./AppCreationScripts/AppCreationScripts.md)

1. Open the Visual Studio solution and click start

If ou don't want to use this automation, follow the steps below:

#### Choose the Azure AD tenant where you want to create your applications

As a first step you'll need to:

1. Sign in to the [Azure portal](https://portal.azure.com) using either a work or school account or a personal Microsoft account.
1. If your account is present in more than one Azure AD tenant, select `Directory + Subscription` at the top right corner in the menu on top of the page, and switch your portal session to the desired Azure AD tenant.   
1. In the left-hand navigation pane, select the **Azure Active Directory** service, and then select **App registrations**.

#### Register the client app (up-console)

1. In **App registrations** page, select **New registration**.
1. When the **Register an application page** appears, enter your application's registration information:
   - In the **Name** section, enter a meaningful application name that will be displayed to users of the app, for example `up-console`.
   - In the **Supported account types** section, select **Accounts in any organizational directory**.
    > Note that if there are more than one redirect URIs, you'd need to add them from the **Authentication** tab later after the app has been created succesfully. 
1. Select **Register** to create the application.
1. On the app **Overview** page, find the **Application (client) ID** value and record it for later. You'll need it to configure the Visual Studio configuration file for this project.
1. In the list of pages for the app, select **Manifest**, and:
   - In the manifest editor, set the ``allowPublicClient`` property to **true** 
   - Select **Save** in the bar above the manifest editor.
1. In the list of pages for the app, select **API permissions**
   - Click the **Add a permission** button and then,
   - Ensure that the **Microsoft APIs** tab is selected
   - In the *Commonly used Microsoft APIs* section, click on **Microsoft Graph**
   - In the **Delegated permissions** section, ensure that the right permissions are checked: **User.Read**, **User.ReadBasic.All**. Use the search box if necessary.
   - Select the **Add permissions** button

1. At this stage permissions are assigned correctly but the client app does not allow interaction. 
   Therefore no consent can be presented via a UI and accepted to use the service app. 
   Click the **Grant/revoke admin consent for {tenant}** button, and then select **Yes** when you are asked if you want to grant consent for the
   requested permissions for all account in the tenant.
   You need to be an Azure AD tenant admin to do this.

### Step 3:  Configure the sample to use your Azure AD tenant

In the steps below, "ClientID" is the same as "Application ID" or "AppId".

Open the solution in Visual Studio to configure the projects

#### Configure the client project

> Note: if you used the setup scripts, the changes below will have been applied for you

1. Open the `up-console\appsettings.json` file
1. Find the app key `ClientId` and replace the existing value with the application ID (clientId) of the `up-console` application copied from the Azure portal.
1. (Optionally) Find the line where `Tenant` is set and replace the existing value with your tenant ID.

### Step 4: Run the sample

Clean the solution, rebuild the solution, and start it in the debugger.


## Community Help and Support

Use [Stack Overflow](http://stackoverflow.com/questions/tagged/msal) to get support from the community.
Ask your questions on Stack Overflow first and browse existing issues to see if someone has asked your question before.
Make sure that your questions or comments are tagged with [`msal` `dotnet`].

If you find a bug in the sample, please raise the issue on [GitHub Issues](../../issues).

To provide a recommendation, visit the following [User Voice page](https://feedback.azure.com/forums/169401-azure-active-directory).

## Contributing

If you'd like to contribute to this sample, see [CONTRIBUTING.MD](/CONTRIBUTING.md).

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information, see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## More information

For more information about the app registration:

- [Microsoft identity platform](https://aka.ms/aaddevv2)
- [Quickstart: Register an application with the Microsoft identity platform](https://docs.microsoft.com/azure/active-directory/develop/quickstart-register-app)
- [Quickstart: Configure a client application to access web APIs](https://docs.microsoft.com/azure/active-directory/develop/quickstart-configure-app-access-web-apis)

For more information, see MSAL.NET's conceptual documentation:

- [MSAL.NET's conceptual documentation](https://aka.ms/msal-net)
- [Customizing Token cache serialization](https://aka.ms/msal-net-token-cache-serialization)
- [Scenarios](https://aka.ms/msal-net-scenarios)
- [Acquiring Tokens](https://aka.ms/msal-net-acquiring-tokens)
- [Username/password](https://aka.ms/msal-net-up)
- [Microsoft identity platform and the OAuth 2.0 resource owner password credential](https://review.docs.microsoft.com/en-us/azure/active-directory/develop/v2-oauth-ropc) to learn more about the underlying protocol
- [Resource owner password credentials RFC](https://tools.ietf.org/html/rfc6749#section-1.3.3)

For more information about the Microsoft identity platform see:

- [Microsoft identity platform](https://aka.ms/aaddevv2)
