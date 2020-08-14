---
layout: post
title:  "Automating Azure Automation Accounts - Part 2 - Creating the custom module"
date:   2020-07-15  20:00:00 -0500
categories: Azure Automation DevOps Module
---

Now that the manual stuff was out of the way, it's time to set up the custom PowerShell module.

<!--more-->

# Part 2 - Setting up a PowerShell Module for Automation Account

## Introduction:
This article is part of a series on integrating Azure Automation Accounts with Azure DevOps. You can check out [Part 1](./AutomatingAutomationAccounts-Part-1.md) first if you'd like.

## Problem:
Following up from my previous post, I now have these things set up:
* [x] An Azure DevOps Organization
* [x] An Azure DevOps Repository (`AutomationRunbooks`)
* [x] An Azure Automation Account 
* [x] Source control configured to pull from the repo

I can't stand having to keep track of multiple copies of code (the [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) principal), so I needed a way to share code between multiple runbooks. In order to accomplish this, I decided to create a PowerShell Module and use that to share code *([additional notes below](#caveats))* across runbooks.

## Solution 

Creating a custom module seemed easy, but there's some nuance to it when you have to deploy in an Automation Account. I loosely followed a few articles like [this one](https://blog.kloud.com.au/2018/05/23/creating-your-own-powershell-modules-for-azure-automation-part-1/) when I was started working on this, but these are the basic steps I took:

1. Open [VSCode](https://code.visualstudio.com/) and use the terminal (with PowerShell) to navigate to your desired directory
2. Create and navigate to a new folder for your module <br/>`md AutomationModule; cd AutomationModule;`
3. Create a new module manifest file (and open it) <br/>`New-ModuleManifest -Path .\AutomationModule.psd1; code .\AutomationModule.psd1;`
4. Edit the manifest as follows:
   * Replace `RootModule` (~line 12) with `RootModule = 'AutomationModule.psm1`
   * Replace `FunctionsToExport` (~line 72) with `FunctionsToExport = '*'` 
   <br/>*The original value caused issues when I uploaded the module and imported it*
   * Replace `CmdletsToExport` (~line 72) with `CmdletsToExport = '*'`
   * Replace `AliasesToExport` (~line 72) with `AliasesToExport = '*'`
   * *Update other properties as desired*
5. Create a new module script
<br/>`code AutomationModule.psm1`
   * I used what I found to be a common style of importing/exporting functions in a module (the *Public/Private* method)
   ```PowerShell
    Write-Verbose "Loading AutomationModule";

    # Load Private functions 
    ## These are internal to the module and can't be accessed outside of it
    $(Get-ChildItem -Path $PSScriptRoot\Private\*.ps1) | %{ . $_.FullName; }

    # Load Public functions
    ## These will be loaded and exported as functions available to any script that imports this module
    $(Get-ChildItem -Path $PSScriptRoot\Public\*.ps1) | %{ 
        Write-Verbose "- $($_.FullName) | $($_.BaseName)";
        . $_.FullName; 
        Export-ModuleMember -Function $_.BaseName -Verbose; 
    }
    ```
6. Create *Private* and *Public* folders in the root of your module directory along with any additional functions that serve each of those purposes
  <br/> `mkdir Private; mkdir Public;`
7. Test the module manifest to ensure it's valid
  <br/>`Test-ModuleManifest -Path AutomationModule.psd1;`

Now that the module has been created and tested, I needed to  uploaded it to my Automation Account. In order to do that, I had to perform a few steps:

1. Compress just the module files (ignore zip file if we run this again)
  ```PowerShell
  # Compress all the files in the directory except for any existing zip file we've already created for this module
  Get-ChildItem -Path . -Exclude @('AutomationModule.zip') |Compress-Archive -DestinationPath AutomationModule.zip -Confirm:$false -Force;
  ```
2. In the Automation Account, navigate to the Modules section
    1. Select ***Add a module*** and upload the newly created zip file
    2. The modules list will show the uploaded module with a importing status. This may take a few minutes or to import and should be successful


#### Notes : {#caveats}
When uploading a module to the Automation Account, you may notice that the file uploads to an Azure Storage account, after which it's imported. We will need to duplicate this process manually when we set up a deployment pipeline in Azure DevOps.

One big thing to remember is that once a module is uploaded to the automation account, there's **no way to download it** or look at it's contents. You can delete the module or upload a new version, but that's it. Keep that in mind as you consider what code you want to include in your module vs directly in your runbooks. My recommendation is to only put shared (& well tested) code in a module. That way you won't need to constantly be uploading new versions and when you're trying to debug your runbooks, you won't be flying blind.


#### Conclusion

To wrap up *Part 2*, we'll review what we've done:
* [x] Create a new PowerShell Module in VSCode
* [x] Upload the module to an Automation Account

In **Part 3**, I'll show you how to automate the deployment of the module to an automation account any time a new commit is made to the *master* branch.

#### Links
* [Intro  - Introduction to the series]({% post_url 2020-07-01-AutomatingAutomationAccounts-Intro %})
* [Part 1 - Setting up the Automation Account + Source Control]({% post_url 2020-07-08-AutomatingAutomationAccounts-Part-1 %})
* [Part 2 - Setting up the custom Module]({% post_url 2020-07-15-AutomatingAutomationAccounts-Part-2 %})
* [Part 3 - Automating Custom Module Deployment to Automation Account]({% post_url 2020-07-22-AutomatingAutomationAccounts-Part-3 %})
