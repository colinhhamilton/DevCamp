# DevOps with Visual Studio Team Services (NodeJS)

## Overview
In this lab, you will create a Visual Studio Team Services online account, check in your code, create a Continuous Integration pipeline, and test your cloud-based application.

## Objectives
In this hands-on lab, you will learn how to:
* Create a Visual Studio Team Services online account
* Create a VSTS Git repository
* Add your code to the VSTS Git repository
* Create a Continuous Integration pipeline

## Prerequisites

* The source for the starter app is located in the [start](start) folder. 
* There will be no code changes required so the the `end` folder will remain empty. 
* Deployed the starter ARM Template [HOL 1](../01-developer-environment)
* Completion of the [03-azuread-ofice365](../03-azuread-office365)  

## Exercises
This hands-on-lab has the following exercises:
* Exercise 1: Create VSTS online account 
* Exercise 2: Create VSTS Git repository
* Exercise 3: Add application to VSTS Git
* Exercise 4: Create a Continuous Integration pipeline
* Exercise 5: Deploy code to an Azure Web App

---
### Exercise 1: Create VSTS online account

1. In your browser, browser to [https://www.visualstudio.com/team-services/](https://www.visualstudio.com/team-services/)

    ![image](./media/image-000.gif)

1. Log in with your account 

---
## Exercise 2: Create VSTS Git repository

VSTS gives us the option to use Git or [TFVC](https://www.visualstudio.com/en-us/docs/tfvc/overview) as our project's repository.  For this exercise we will use Git, and then clone the repository to our dev machine. 

> Note that if you acquired these lab materials via a `git clone` of the workshop repo then you should select a folder somewhere else on your dev machine. This will minimize conflicts between the two separate repositories 

1. Starting at your account's landing page, locate the section entitle **Recent projects & teams** and click **New**.

    ![image](./media/image-001.gif)

1. Enter a project name such as **DevCamp**, ensure **Version Control** is set to **Git** and then click **Create Project**.

    ![image](./media/image-002.gif)

1. Wait for the project to be created. This process may take up to 60 seconds. When finished select the **Navigate to Project** button

    ![image](./media/image-003.gif)

1. Exit out of the Congratulations window and explore your pre-built dashboard. Familiarize yourself with the variety of widgets available, and the customization options. 

    ![image](./media/image-004.gif)

1. Click **Code** on the top toolbar to navigate to the Code screen.  Then click the **Generate Git Credentials** button to set a user name, alias, and password.

    ![image](./media/image-005.gif)

1. Next, select the **Copy** icon to copy the HTTPS URL for the repository.

1. In a console window, navigate to a spot on your dev machine and execute the following (Replace the value for your repo):

    ```CMD
    git clone https://[yourvstsrepo].com/DefaultCollection/_git/Repo.git    
    ```

    ![image](./media/image-006.gif)

    > Depending on your environment setup you may need to authenticate with VSTS

You have now created a project in VSTS with a Git repository, and cloned the repository locally to your developer machine.  Next we'll upload code from our machine to VSTS.

---
## Exercise 3: Add application to VSTS Git

1. When we cloned our repository it was empty.  Take the code that you have developed in the earlier labs (or the `start` folder bundled with this readme) and paste it into our new directory.  This can be done via the command line, or with good old copy/paste in an Explorer or Finder window.

    ![image](./media/image-007.gif)

    > Depending on how your environment is setup, there may be a hidden folder `.git` in your originating directory. Do not copy this folder into the destination directory linked to VSTS

1. Back in the console, execute a `git status` to ensure the files are picked up by git.

    ```CMD
    git status    
    ```

    ![image](./media/image-008.gif)

1. Execute `git add *` to track the files, then a `git commit -m "initial upload"` to commit the files to the repository. Finally, execute `git push origin master` to push the files up to VSTS.

    ```CMD
    git add *
    git commit -m "initial upload"
    git push origin master
    ```

    ![image](./media/image-009.gif)

1. In the browser, reload the **Code** page to see the uploaded code

    ![image](./media/image-010.gif)

1. Now, any changes you make to the local repository can be pushed up to VSTS.  Other team members may also begin interacting with the code base via their own clones and pushes.

> Note that we did not include the `node_modules` or `.vscode` folders. These components are typically not added to source control, as they bloat the size of the repository.  These files should have been excluded from your repository due to settings in the `.gitignore` file

---
## Exercise 4: Create a Continuous Integration pipeline

With application code now uploaded to VSTS, we can begin to create builds via a Build Definition.  Navigate to the **Build** tab from he top navigation.  We will use the hosted agent within VSTS to process our builds in this exercise.

1. From the **Build** tab, create a new **Build Definition**

    ![image](./media/image-011.gif)

1. There are prebuilt definitions for a variety of programming languages and application stacks, however for this exercise select **Empty** and click **Next**

    ![image](./media/image-012.gif)

1. Confirm the Repository Source is set to your VSTS Project, that the repository is set the repo that was earlier created, and that the Agent Queue is set to **Hosted**.  

1. Check the box next to **Continuous Integration** to automatically run this build anytime code is checked into the repository.

    ![image](./media/image-013.gif)

1. After the empty Build Definition is created, we need to create a series of Build Steps.

    * Verify NodeJS version installed on the build agent by echoing it to the console
    * Restore all package dependencies with `npm install`
    * Package the code assets into a deployable zip file
    * Publish the zip file as a Publish Artifact that can be consumed by the VSTS Release System

    Each of these steps will begin by clicking the **Add build step** button

    ![image](./media/image-014.gif)

1.  Add a Build Step for **Command Line**, found under the left-hand filter for **Utility**

    ![image](./media/image-015.gif)

1. Configure the step **Tool** to `node` and the **Argument** to `-v`

1. Click the pencil icon to name this build step to **Echo Node Version**

    ![image](./media/image-017.gif)

1. Add a Build Step for **npm**, found under the left-hand filter for **Package**

    ![image](./media/image-018.gif)

1. Configure **Command** for `install` and name the step **Install Dependencies**

    ![image](./media/image-019.gif)

    > If you are using `devDependencies` in your `package.json` and want to control whether or not they are installed, then pass the `--production` flag in the **Argument** field  

1. Add a Build Step for **Archive**

    ![image](./media/image-020.gif)

1. In configuration boxes, we can use variables in addition to string literals.  Configure **Root Folder** to use the directory on the Build Agent that contains our sources files by inserting `$(Build.SourcesDirectory)`. 
    
1. For **Archive file to create** insert `$(Build.SourcesDirectory)/archive/$(Build.BuildId).zip`. This will dynamically name our zip file of code with the build number.

1. Uncheck the box for **Prefix root folder name to archive paths** to avoid an unnecessary nesting within the .zip file.

    ![image](./media/image-021.gif)

    > You can define your own variables to use throughout the Build and Release pipelines by clicking **Variables** in the Build Definition's sub-navigation. Also see [here](https://www.visualstudio.com/docs/build/define/variables) for all pre-defined variables available 

1. Finally, create a Build Step for **Publish Build Artifacts**.  This step outputs a file(s) from our Build Definition as a special "artifact" that can be used in VSTS' Release Definitions.

    ![image](./media/image-022.gif)

1. Configure **Path to Publish** as `$(Build.SourcesDirectory)/archive/$(Build.BuildId).zip` to target the zip file created in the previous Build Step.

    > For **Artifact Name** enter `drop`
    >
    > Set **Artifact Type** to `Server`
    >

    ![image](./media/image-023.gif)

1. Save your Build Definition named **BuildApp**

    ![image](./media/image-016.gif)

1. Our saved Build Definition is ready to be processed by the Hosted Build Agent.  Click **Queue New Build** to start the build process. 

    ![image](./media/image-024.gif)

1. Accept the defaults and click **OK**

    ![image](./media/image-025.gif)

    Your Build will then be queued until the Hosted Build Agent can pick it up for processing.  This typically lasts less than 60 seconds to begin.

1. Once your Build completes, click each step on the left navigation bar and inspect the output.  For **Echo Node Version** we can see the agent's version in the right **Logs** pane

    ![image](./media/image-026.gif)

1. Let's inspect the output artifacts that were published.  Click the **Build 213** header in the left pane to view the build's landing page.  Then select **Artifacts** from the horizontal toolbar, and **Download** the **drop** artifact.

    ![image](./media/image-027.gif)

1. Unzip `drop.zip` to see our files (including the restored `node_modules` folder).  This artifact will be deployed to an Azure Web App in a later exercise.

    ![image](./media/image-028.gif)

We now have a Build Definition that will construct our NodeJS application and package it for deployment anytime code is checked into the repository, or a manual build is queued. 

---
## Exercise 5: Deploy code to an Azure Web App

In the ARM Template that was originally deployed in the lab setup, a web app was created as a development environment to hold a NodeJS application. We will use this web app as a deployment target from VSTS. First, we need to prepare this web app for our application code and then create a Release Definition.

1. Visit the Azure Web App by browsing to the [Azure Portal](http://portal.azure.com), opening the Resource Group, and select the Azure Web App resource that beings **nodejsapp** before the random string. 

    ![image](./media/image-029.gif)

1. Once the blade expands, select **Browse** from the top toolbar

    ![image](./media/image-030.gif)

    A new browser tab will open with a splash screen visible

    ![image](./media/image-031.gif)

1. We can deploy our code to this Azure Web App, however it was not configured with our AzureAD details. When trying to authenticate, AzureAD would refuse since it does not know about this domain. 

    To fix this, return to `https://apps.dev.microsoft.com`, login, and open your application settings. 

    ![image](./media/image-032.gif)

1. In the section for **Platforms**, click **Add Url** to add the URL of your Azure Web App from Step 1.  Remember to append the `/auth/openid/return` route at the end, since that is the route that will process the return data from AzureAD. Ensure this address is using **https**.

    ![image](./media/image-033.gif)

1. Make sure you click **Save** at the bottom of the screen to add the URL to your AzureAD app.

1. Now that AzureAD is configured, we need to add our AzureAD related environment variables to the Azure Web App.  Back in the **nodejsapp** blade where you hit **Browse** earlier, open **Application Settings** from the left navigation.

    ![image](./media/image-034.gif)

1. Find the **App Settings** section containing a table of settings.  In the ARM Template we auto-generated the majority of these settings, however we need to add a few additional environment variables to match the `.vscode/launch.json` file that we have been using locally.

    * **AAD_RETURN_URL** should be set to the same URL that we just configured for our AzureAD application. Should be similar to `https://nodejsappmm6lqhplzxjp2.azurewebsites.net/auth/openid/return`. Ensure this is using **https**.

    * **AAD_CLIENT_ID** should match launch.json and similar to `2251bd08-10ff-4ca2-a6a2-ccbf2973c6b6`

    * **AAD_CLIENT_SECRET** should match launch.json and be similar to `JjrKfgDyo5peQ4xJa786e8z`

    ![image](./media/image-035.gif)

1. Now that the AzureAD application and the Azure Web App are ready, let's configure VSTS to deploy our built application with a Release Definition. Back in  VSTS, select the **Build & Release** from the top navigation, and click **Releases**. 

    ![image](./media/image-044.gif)

1. Click the **New definition** button

    ![image](./media/image-045.gif)

1. Then select the **Empty** template 

    ![image](./media/image-046.gif)

1. Ensure the **Source** is set to the Build Definition name used in the earlier exercise and that **Queue** is set to the **Hosted** option. Then click **Create** to finish creating the Release Definition

    ![image](./media/image-047.gif)

1. Click on the Pencil icon next to the Release Definition name and rename it to **ReleaseApp**.  Then click into **Environment 1** and name it **Dev**

    ![image](./media/image-048.gif)

1. For the newly renamed Dev environment, click **Add tasks** then from the **Deploy** tab on the left navigation choose **Azure App Service Deploy**.

    ![image](./media/image-036.gif)

    > Avoid the similarly named `Azure App Service Classic (Deprecated)` task, as it uses the older ASM Model rather than [ARM](https://docs.microsoft.com/en-in/azure/azure-resource-manager/resource-group-overview#the-benefits-of-using-resource-manager)

1. VSTS needs a connection to a target Azure Subscription. Click **Manage** to open a new tab holding configuration options.

    ![image](./media/image-037.gif)

1. In the new tab, select **New Service Endpoint** and from the dropdown choose **Azure Resource Manager**

    ![image](./media/image-038.gif)

1. The modal window should automatically determine your subscription information.  Provide a name such as **Azure**, select **OK**, and skip the remaining instructions for this step.

    ![image](./media/image-039.gif)

    However, if your subscription is not in the dropdown list, then VSTS could not automatically configure a Service Principal for your account and you will need to manually create the SP. A Service Principal is similar to a service account, in that it is a separate account within Azure Active Directory that is given permissions to activities in an Azure Subscription (such as the creation or deletion of Azure resources). 

    To begin manually creating a Service Principal, click the **here** link at the bottom of the "Add Azure Resource Manager Service Endpoint" modal window.   

    ![image](./media/image-043a.gif)

    The window format will change to allow you to enter connection information on your subscription. 

    ![image](./media/image-054.gif)

    For **Connection Name**, enter a value of `Azure`.

    **Subscription ID** is the GUID for your Azure Subscription. This can be found by opening the [Subscriptions Blade](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade) from the Azure Portal, and selecting your subscription. The GUID will be in the Subscription Details blade. Next to the ID is the value for **Subscription Name**. Copy these values back in the Modal Dialog in VSTS.

    ![image](./media/image-052.gif)

    To create a Service Principal, open a terminal window that has the [AzureCLI installed](https://docs.microsoft.com/en-us/azure/xplat-cli-install), and execute `azure ad sp create -n DevCampSP -p Devc@mp2016!`. The returned **Service Principal Name** maps to **Service Principal Client ID** in VSTS.  The password used (`Devc@mp2016!`) maps to the **Service Principal Key** in VSTS.  Also take note of the returned **Object ID** value for the next step.

    Next, we need to grant the new SP "Contributor" permissions for our Azure subscription. Execute the following command, substituting the content between `<>` for the Object ID returned from the SP creation command, and the Subscription ID returned from the Azure Portal Blade.
    
    ```shell
    azure role assignment create -o Contributor --objectId <Object ID returned in previous step> -c /subscriptions/<Subscription ID GUID retrieved earlier from portal>`
    ```

    An example command looks like: 

    ```shell
    azure role assignment create -o Contributor --objectId ae5350b5-2346-4509-8184-d83f296d3cac -c /subscriptions/9f4d814b-7085-44ae-0f99-bfh8sf5a3f35
    ```

    The **Tenant ID** in VSTS means the GUID of your Azure Active Directory tenant.  To find the value, open the [Properties Blade](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Properties) in the Azure Portal's AAD Blade.

    To visualize where each Service Endpoint value is found, please see:
    ![image](./media/image-053.gif)
    
    Once each value is filled out, click **Verify Connection** to ensure the values work, then click **OK** to finish creating the Service Endpoint connection to Azure.

    ![image](./media/image-048a.gif)

    > This Service Endpoint pattern is used to connect to a variety of services beyond Azure such as Jenkins, Chef, and Docker

1. Back on the VSTS Build window, in the Build Step we started earlier, click the **Refresh** icon. The **Azure** connection that we setup should now appear.  Select it. 

    ![image](./media/image-040.gif)

1. Next, for **App Service Name** choose the name of the Node Azure Web App. It may take a moment to populate.

    ![image](./media/image-041.gif)

1. **Save** the Release Definition, and **Create Release**

    ![image](./media/image-042.gif)

1. Select the latest completed Build, and then click **Create**

    ![image](./media/image-049.gif)

1. Click on the **Release 1** link to view details of the release
   
    ![image](./media/image-050.gif)

1. On the release details screen, click **Logs** from the top toolbar to get details about each release step and following the release progress

    ![image](./media/image-051.gif)

1. After a successful release you should see the application deployed to your web app

    ![image](./media/image-043.gif)

---
## Summary

In this hands-on lab, you learned how to:
* Create a Visual Studio Team Services online account
* Create a VSTS Git repository
* Add your code to the VSTS Git repository
* Create a Continuous Integration pipeline
* Deploy a built application to an Azure Web App from VSTS

---
Copyright 2016 Microsoft Corporation. All rights reserved. Except where otherwise noted, these materials are licensed under the terms of the MIT License. You may use them according to the license as is most appropriate for your project. The terms of this license can be found at https://opensource.org/licenses/MIT.
