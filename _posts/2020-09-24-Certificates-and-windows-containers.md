---
title: Installing the certificate in root store for windows container
tags: [Windows Container, Docker, Certificates, Root Certificate]
style: border 
color: light 
description: In this post, we will look at the error and solution that we see when trying to install a certificate in root store for a windows container.
---

I had to connect to a remote address from within a container and the remote server was using a [Private Certificate Authority](https://searchsecurity.techtarget.com/definition/private-CA-private-PKI). For a successful and secure connection with the remote machine, the code running in windows container had to trust the remote certificate. 

When trying to add a certificate at runtime, the following code was throwing an error that reads "unable to open a store" or "access denied".

