---
title: Mystery of WMI consuming most CPU when under load
tags: [Software performance optimization, .net performance trick, WMI, high CPU]
style: border 
color: light 
description: When running performance simulations, we saw that suddenly WMI (Windows management instrumentation) was taking an unusually high amount of CPU. We initially thought that it is an environmental issue but it turned out to be an innocent-looking line in our code. Read the post to understand how we investigated the problem and reached to a fix. 
---

A teammate told me that when running performance simulations on our product such that the load is of a few thousand users, the WMI windows service takes up all the CPU and the tests does not scale. The first thought was to open a case with Microsoft to see if they see anything in the operating system that might cause the problem. The conversations did not give us much clue. 

I decided to dig in further to see what the WMI windows service is doing. A lot of you might not know but the WMI windows service can write the details of the query to event viewer (similar to SQL profiler). 

...work inprogress
