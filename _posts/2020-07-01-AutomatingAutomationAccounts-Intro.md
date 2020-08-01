---
layout: post
title:  "Automating Azure Automation Accounts - Introduction"
date:   2020-07-01  20:00:00 -0500
categories: Azure Automation DevOps Module
---

Welcome to my short blog series on *Automating the deployment of an Azure Automation Account*. This series is primarily a brain dump of my thoughts as I've been working through this process. I'll be discussing `runbooks` as well as deploying custom modules via `Azure DevOps`.

<!--more-->

#### Series Table of Contents

* [Intro  - Introduction to the series]({% post_url 2020-07-01-AutomatingAutomationAccounts-Intro %})
* [Part 1 - Setting up the Automation Account + Source Control]({% post_url 2020-07-08-AutomatingAutomationAccounts-Part-1 %})
* [Part 2 - Setting up the custom Module]({% post_url 2020-07-15-AutomatingAutomationAccounts-Part-2 %})


#### Introduction:
Scheduled tasks in the cloud just aren't the same as they were on-premises. It's not just throw a script on the file system, open Task Scheduler and add a job. Or is it? One of the challenges that I've had as my organization has moved to the cloud is figuring out the best way to handle scheduled tasks. 

When we first started shifting to the cloud, I though to myself that *"this will be great. I'm sure there's an easy way to take my tasks, run them in the cloud and we get to retire another on-prem server"*. It would've been great if only it were that easy. What I found as I peeled back the first layer of the onion is that you have to *think differently in the cloud*. Now, that's not a novel thought. It's not some deep revelation that's going to change the world. Yet, it is a revelation that every person has to go through in their cloud transformation journey. If you don't come to the realization, **you risk migration without transformation**. And that can be a very costly decision in time and money. The cloud doesn't naturally reduce complexity or cost or FTE needs (or enhance security for that matter). It does however, change them. Sure, you're not waking up in the middle of the night rushing to the office because a power supply failed on your server, but there will be other emergencies you'll need to prepare for.

That cloud perspective is where I initially got hung up. I understood the idea of the cloud. SaaS, PaaS & IaaS were buzzwords. Serverless was the cool kid on the block now. I even played around a bit with all these things. But I still never got around to *pulling the cruft* into my transformation strategy and replatforming my scheduled tasks to fit the cloud model. 

So as I have begun rethinking my schedule tasks, I wanted to leverage [Azure Automation Accounts](https://azure.microsoft.com/en-us/services/automation/) for a few of its features - like its *logging*, *scheduling* and *[hybrid connector](https://docs.microsoft.com/en-us/azure/automation/automation-hybrid-runbook-worker)* features.

---

I'll continue updating this page as I roll out new articles, but here's the jist:

#### Part 1 - Preparing the platforms
* Ensure you have a DevOps account set up
* Create a new DevOps repo
* Create a new Automation Account
* Create runbooks, modules and secrets
* Connect it up to DevOps for runbook source control

#### Part 2 - Setting up a PowerShell Module for Automation Account

#### Part 3 - Automating custom module deployment

---

### Issues:

Occasionally I'll run into issues and try to track them all here as well as in the original articles too, just for visibility

***Unable to import Python runbook***

Even though the documentation mentions you can use Python runbooks, when I tried, I kept getting an error. No way to overcome it except to remove the pythong runbooks. `#bug`

```
Failed to import runbook 'Test.py'.
Exception: Cannot validate argument on parameter 'Type'. The argument "Python2" does not belong to the set "Graph,PowerShell,PowerShellWorkflow" specified by the ValidateSet attribute. Supply an argument that is in the set and then try the command again.
Failed to sync. 
Failed to import runbook:
- Test.py
```