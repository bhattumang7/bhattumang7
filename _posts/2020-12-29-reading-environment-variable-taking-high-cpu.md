---
title: Mystery of WMI consuming most CPU when under load
tags: [Software performance optimization, .net performance trick, WMI, high CPU]
style: border 
color: light 
description: When running performance simulations, we saw that suddenly WMI (Windows management instrumentatio) was taking unusually high amount of CPU. We initially thought that it is an environment issue but it turned out to be an innocent looking line in our code. Read the post to understand how we investigated the problem and reached to a fix. 
---
