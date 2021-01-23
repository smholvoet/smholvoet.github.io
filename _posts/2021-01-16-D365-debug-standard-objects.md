---
published: true
title: "Debug standard objects in Dynamics 365"
excerpt: "Debugging standard objects in Dynamics 365 F&O"
date: 2021-01-16T00:00:00-04:00
show_date: true
tags:
  - D365
  - F&O
  - Finance & Operations
  - SCM
  - Dynamics
  - breakpoint
  - debugging
---

The below steps will show you how to use the built-in debugger and set breakpoints in any of the standard objects within D365 F&O.

## Symbol settings

Navigate to **Tools > Options > Debugging > Symbols** and check **Load all modules, unless excluded**:

![Debugging settings 1](/assets/images/\debugging-settings-2.png)

Keep the same *Options* window open and navigate to **Dynamics 365 > Debugging**. Make sure **Load symbols only for items in the solution** is unchecked:

![Debugging settings 2](/assets/images/\debugging-settings-1.png)

A general explanation surrounding the use of symbol files within *Visual Studio* can be found [here](https://docs.microsoft.com/en-us/visualstudio/debugger/specify-symbol-dot-pdb-and-source-files-in-the-visual-studio-debugger?view=vs-2019).

## Set a breakpoint

Lets add a breakpoint to a standard object, for example the `run` method of the `CustTable` form:

![CustTable](/assets/images/\custtable.png)

Add the 🔴 [breakpoint](https://docs.microsoft.com/en-us/visualstudio/debugger/debugger-feature-tour?view=vs-2019#set-a-breakpoint-and-start-the-debugger) (`F9`):

![Add breakpoint](/assets/images/\add-breakpoint.png)

## Attach to process

Navigate to **Debug > Attach to Process**, which will open the *Attach to Process* dialog box:

!Attach to process 1](/assets/images/\attach-to-process.png)

![Attach to process 2](/assets/images/\attach-to-process-2.png)

- **Connection type:** leave *Default* selected
- **Connection target:** hostname of your local machine

At the bottom, make sure to select **Show processes from all users**. In the list of *Available processes*, search for the `w3wp.exe` process. This is the [IIS](https://docs.microsoft.com/en-us/iis/get-started/introduction-to-iis/iis-web-server-overview) worker process, the local web server hosting our F&O instance.

⚠️ If you're using [IIS Express](https://docs.microsoft.com/en-us/iis/extensions/introduction-to-iis-express/iis-express-overview) instead of *IIS*, search for the {% raw %}`iisexpress.exe`{% endraw %} process.
{: .notice--info}

After selecting the correct process, click **Attach**. This will start the debugger within Visual Studio.

## Launch browser

Now launch F&O from your browser and call the `CustTable` form by going to **Accounts receivable > Customers > All customers**:

![All customers](/assets/images/\all-customers.png)

*Visual Studio* will then hit the breakpoint upon loading the form:

![Hit breakpoint](/assets/images/\hit-breakpoint.png)

More information on debugging within D365 F&O can be found on 📖 [Microsoft Docs](https://docs.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/dev-tools/build-debug-project).
