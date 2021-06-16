---
title: "Get Max File Size for Azure Storage Container"
excerpt: >-
  When you need to get the max file size for a container in Azure storage, just use the Azure CLI.
categories:
  - Azure
  - CLI
  - PowerShell
  - Snippet
tags:
  - Azure
  - CLI
  - PowerShell
  - Identity
date:   2021-06-16  13:00:00 -0500
---

Recently, I got a request from a coworker to determine the maximum file size we have in an Azure Storage container. This seemed simple enough, but I didn't find a quick solution, so I decided to publish something myself. I had 2 criteria to fulfill for the solution. First, I wanted to use the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/) (I don't use it as often as I'd like, so I try to find oportunities when I can). The second was to use a [JMESPath](https://jmespath.org/) query. Because I don't use either very often, it took a few minutes to get the syntax just right. 

The way I did all this was through the [Azure Cloud Shell](https://docs.microsoft.com/en-us/azure/cloud-shell/overview) via [Windows Terminal](https://github.com/microsoft/terminal).

And here's the snippet:

```powershell
$b = $(az storage blob list --account-name STORAGE_ACCOUNT_NAME --container-name CONTAINER_NAME --query "max([].properties.contentLength)"); 
$mb = $b/1Mb;
Write-Host "Largest file: $($mb)Mb ($($b) bytes)";
```

You can always use `az interactive` to help with syntax on the CLI commands, as it will provide some intellisense. I find it very helpful for getting things going quickly. The JMESpath queries can be a bit hit or miss, and I have to remind myself of the syntax everytime. Thankfully, the [website](https://jmespath.org/examples.html) has some good examples.