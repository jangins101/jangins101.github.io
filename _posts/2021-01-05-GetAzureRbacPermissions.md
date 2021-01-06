---
title: "Get RBAC Permission for Azure Resources"
excerpt: >-
  Sometimes you need a quick script to pull all the permissions for a subscription, management group, resource group, or on a specific resource.
categories:
  - Azure
  - PowerShell
  - Security
tags:
  - Azure
  - PowerShell
  - Identity
  - Security
  - RBAC
date:   2021-01-05  13:00:00 -0500
---

Have you ever just needed to grab a quick report of the RBAC roles and permissions in your Azure environment? Maybe you're working on a [Resource Move](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/move-resource-group-and-subscription) across subscriptions, or performing a _security audit_. No matter the reason, there is a quick and easy way to pull the information you want. 

Below you will find a simple snippet that will grab each of the subsriptions in your Azure tenant, loop through them and save a csv of the permissions found. This is done using the [Get-AzRoleAssignment](https://docs.microsoft.com/en-us/powershell/module/az.Resources/Get-azRoleAssignment?view=azps-5.3.0) cmdlet. 

Note that this will list all the unique permissions within the scope (inherited permissions will not show up at each level). This is helpful because it minimizes the number of records returned.

_**tl;dr:** Run `Get-AzRoleAssignment` once connected to Azure to grab the RBAC permissions for the selected subscription scope._

```powershell
# Load the required modules
Import-Module Az;
Import-Module Az.Resources;

# Connect to Azure
Connect-AzAccount;
Write-Host "Connected to Azure";

# Grab all the subscriptions in the tenant
# Note: this will include the AzureAD subscription, but it will error out as it doesn't support listing the RBAC roles this way
$subs = Get-AzSubscription;
Write-Host "Checking $($subs.Count) subscriptions";

# Loop through each subscription
$rbac = @();
foreach ($sub in $subs) {
    Write-Host "  $($sub.Name) ($($sub.Id)) " -NoNewline;
    # Set the subscription as the current scope
    Select-AzSubscription -Subscription $sub | Out-Null;
    # Grab all the role assignments for this subscription (this will include any management groups as well)
    $tRbac = @(Get-AzRoleAssignment -ErrorAction SilentlyContinue);
    Write-Host " | $($tRbac.Count) unique permissions" -ForegroundColor Yellow;

    $rbac += $tRbac;
}
Write-Host "Total permission: $($rbac.Count)";

# Display the list of role permissions in a gridview
$rbac | Select * | Out-GridView;

# Export the list to a CSV and open it
$outFile =  Join-Path ([System.IO.Path]::GetTempPath()) 'AzRoleAssignments.csv';
$rbac | Export-Csv -Path $outFile -Append;
Invoke-Item $outFile;
```

If you're looking for a simpler version of the code, you can do something like this

```powershell
Connect-AzAccount;
$outFile =  Join-Path ([System.IO.Path]::GetTempPath()) 'AzRoleAssignments.csv';
foreach ($sub in (Get-AzSubscription)) {
  Select-AzSubscription -Subscription $sub | Out-Null;
  @(Get-AzRoleAssignment -ErrorAction SilentlyContinue) | Export-Csv $outFile -Append;
}
ii $outFile;
```