---
#layout: post
title: "How to export all Azure resources to Excel"
excerpt: >-
  Ever wonder how to *export* a list of all your *Azure resources* to JSON, CSV, TSV, YAML or any other format? `n
  Well, here you go, using the latest `Az` module for PowerShell
categories:
  - Azure
  - PowerShell
  - Infrastructure
  - Portal
  - Excel
  - Resources
  - PowerShell
  - Tips
tags:
  - Azure
  - Excel
  - Export
  - PowerShell
  - Az Module
date:   2020-10-12  13:00:00 -0500
categories: Azure Infrastructure Portal Excel Resources PowerShell Tips
---

## How to export all Azure resources to Excel (or CSV, TSV, YAML, JSON, ...)

Sometime you need to review all your resources in Azure. Maybe you have *multiple subscriptions* or one of your teams needs to *identify and clean up* their deployments. No matter the reason, it's helpful to have a quick and easy way to export that list to a file.

We're going to take a look at the [Az Module](https://docs.microsoft.com/en-us/powershell/azure/new-azureps-module-az?view=azps-4.7.0) today and see how easy it is to get this done (see the docs for installation / upgrade procedures).

### Connect to Azure
The `Connect-AzAccount` is used to connect to Azure. The default authentication method will give you a code and have you enter it at [https://microsoft.com/devicelogin](https://microsoft.com/devicelogin). 

Once connected, you can get a list of your subscriptions via the `Get-AzSubscription` cmdlet, and you can change the active subscription using the `Select-AzSubscription -Subscription <id or name>` (where the *Subscription* parameter could specify either the name or the id).

### Grab the list of resources
Next, run the `Get-AzResource` cmdlet to pull all the resources associated with that subscription. Note that if you want to filter this list, you can use one of the parameters as well (e.g. by *ResourceGroupName* or a specific *Tag*).

This cmdlet returns an array of objects with a number of attributes that you may not care about. With this in mind, you can select only the properties you'd like to export, like `Get-AzResource | Select-Object Location, ResourceGroupName, Name, Type`

### Convert to CSV, JSON, YAML
You can take the resulting records and pipe them to a number of cmdlets to get the data you're looking for. For example, you could use `| ConvertTo-Csv` or `| ConvertTo-Json` or if you've installed something like the *[powershell-yaml](https://duckduckgo.com/?t=ffab&q=powershell+yaml&ia=web)* module you could use `| ConvertTo-Yaml`. 

From here you could export that data to a file with the `Out-File` cmdlet, like `Out-File c:\temp\AzureResources.csv`. You can run the file directly following by using the `Invoke-Item` (or its alias `ii`) cmdlet. 

I will typically output to CSV and send the data to my clipboard so I can paste it into excel. The full command I'd run (on Windows) in this instance for that is ``Get-AzResource | Select-Object Location, ResourceGroupName, Name, Type | ConvertTo-Csv -Delimiter `t | clip``. <sub>Note: I typically use the `-NoTypeInformation` paramter as well on *ConvertTo-Csv* as it removes the extra first line describing the type name that I generally don't care about</sub>

### Full Script (CSV)
```powershell
Connect-AzAccount;

# Grab the resources, select the desired properties and export it to a CSV
Get-AzResource `
    | Select-Object Location, ResourceGroupName, Name, Type `
    | Export-Csv -Path c:\temp\AzureResources.csv;
# Open the new file
Invoke-Item c:\temp\AzureResources.csv;

##### --------------------------------------- #####

# Other options for converting data
$resources = Get-AzResource | Select-Object Location, ResourceGroupName, Name, Type;

# Convert to tab-delimited string and copy to clipboard
$resources | ConvertTo-Csv -Delimiter `t | clip

# Convert to JSON and save to a file
$resources | ConvertTo-Json | Out-File c:\temp\AzureResources.json;

# Convert to YAML and save to a file
Install-Module powershell-yaml -Scope CurrentUser; # Install for just the current user
Import-Module powershell-yaml;
$resources | ConvertTo-Yaml | Out-File c:\temp\AzureResources.yaml;
```

Hopefully this quick little tip helped you leverage your PowerShell skills and the Az module to get the information you needed.