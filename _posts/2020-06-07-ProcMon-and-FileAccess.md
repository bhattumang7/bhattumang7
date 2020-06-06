---
title: ProcMon and monitoring file access
tags: [ProcMon, Sysinternals, diagnostic tool,debugging]
style: border 
color: light 
description: ProcMon can be used to diagnose the file access related errors. Let us use ProcMon to see what files are being accessed by the application.
---

Once someone contacted me to understand an error. The error read "Run-time error '76' Path Not found". The application developer had written correct error logging and it included error message and error number. The only thing that was not logged was the path that was not being accessed. Here is how the error looked like:

![Run-time error ‘76’ Path Not found](../assets/blog_pictures/2020-06-07-ProcMon-and-FileAccess/Post_ProcMon_app_error.jpg)

