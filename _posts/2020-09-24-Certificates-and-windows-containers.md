---
title: Installing the certificate in root store for windows container
tags: [Windows Container, Docker, Certificates, Root Certificate]
style: border 
color: light 
description: In this post, we will look at the error and solution that we see when trying to install a certificate in root store for a windows container.
---

I had to connect to a remote address from within a container and the remote server was using a [Private Certificate Authority](https://searchsecurity.techtarget.com/definition/private-CA-private-PKI). For a successful and secure connection with the remote machine, the code running in windows container had to trust the remote certificate. 

When trying to add a certificate at runtime, the following code was throwing an error that reads "unable to open a store" or "access denied".

```csharp
	Console.WriteLine("Trusting root CA");
	// Environment variable with format of "Certificates_SFPkg_Code_MyCert1_PFX" will have path to the certificate file
	// This is injected by the service fabric into the container. 
	// See https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-securing-containers for more details.
	
	string certificateFilePath = System.Environment.GetEnvironmentVariable("Certificates_SFPkg_Code_AllscriptsRootCA_PFX");
	string passwordFilePath = Environment.GetEnvironmentVariable($"CERTIFICATES_SFPkg_CODE_AllscriptsRootCA_PASSWORD");

	X509Store store = new X509Store(StoreName.Root, StoreLocation.LocalMachine);
	store.Open(OpenFlags.ReadWrite);
	string password = File.ReadAllLines(passwordFilePath, Encoding.Default)[0];
	password = password.Replace("\0", string.Empty);
	X509Certificate2 cert = new X509Certificate2(certificateFilePath, password);
	store.Add(cert);
	
	store.Close();
```
