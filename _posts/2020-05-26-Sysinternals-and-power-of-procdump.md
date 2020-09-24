---
title: Procdump and the power of first chance exception
tags: [Procdump, Sysinternals, diagnostic tool,debugging]
style: border 
color: light 
description: Procdump and the power of first chance exception.
---

Let's talk about an interesting issue. I was troubleshooting an issue with the application launch (for a win form application). It showed the outer window but the actual login screen did not appear. Interestingly, the application was not showing any exception and it was not writing anything to error/event log. 

Something went wrong for sure, but it did not propagate the error up to the UI or did not write an error to log (if there was an exception). 

One of the handy tools to investigate this type of problem is "[procdump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump)" from Sysinternals. With the help of this tool, we can check and understand if there are any exceptions being thrown in the background. In Windows terminology, it is called [first chance exception](https://devblogs.microsoft.com/devops/understanding-exceptions-while-debugging-with-visual-studio/).

I started the application like below, using Procdump: 
```powershell
Procdump.exe -e 1 -f "" -x C:\dumps MyApp.exe 
```

The above command launched the application and showed the file not found error with the description saying "FileNotFoundException - Could not load file or assembly .....". 

Somehow the application was suppressing the exception. Once we placed the missing DLL mentioned on the error log, everything worked fine. I suggest you look at the Procdump tool's documentation to understand it's features and parameters. 

This is an arrow worth keeping in your quiver. 

