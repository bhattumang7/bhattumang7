---
title: Installing the certificate in root store for windows container
tags: [Windows Container, Docker, Certificates, Root Certificate]
style: border 
color: light 
description: In this post, we will look at the error and solution that we see when trying to install a certificate in root store for a windows container.
---

I had to connect to a remote address from within a container and the remote server was using a [Private Certificate Authority](https://searchsecurity.techtarget.com/definition/private-CA-private-PKI). For a successful and secure connection with the remote machine, the code running in windows container had to trust the remote certificate. 

When trying to add a certificate at runtime from a windows container, the following code was throwing an error that reads "unable to open a store" or "access denied".

```csharp
Console.WriteLine("Trusting root CA");
// Environment variable with format of "Certificates_SFPkg_Code_RootCACert_PFX" will have path
// to the certificate file.
// This is injected by the service fabric into the container. 
// See https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-securing-containers 
// for more details.

string certificateFilePath = System.Environment.GetEnvironmentVariable("Certificates_SFPkg_Code_RootCACert_PFX");
string passwordFilePath = Environment.GetEnvironmentVariable($"CERTIFICATES_SFPkg_CODE_RootCACert_PASSWORD");

X509Store store = new X509Store(StoreName.Root, StoreLocation.LocalMachine);
store.Open(OpenFlags.ReadWrite);
string password = File.ReadAllLines(passwordFilePath, Encoding.Default)[0];
password = password.Replace("\0", string.Empty);
X509Certificate2 cert = new X509Certificate2(certificateFilePath, password);
store.Add(cert);

store.Close();
```

The reason behind the add failing was that by default the containerized application runs under a user named "ContainerUser" who does not have enough rights. To be able to make the code run successfully in windows container, we can make the following change to the application: 

```dockerfile
FROM mcr.microsoft.com/dotnet/core/aspnet:3.1 AS base
EXPOSE 80

FROM base AS final
WORKDIR /app
COPY ./out /app

# below line will make the application run under ContainerAdministrator and 
# certificate add to Root store will work
USER ContainerAdministrator 

ENTRYPOINT ["dotnet", "MyEntryPOint.dll"]
```

However, making the application run under ContainerAdministrator is not the right way to run the application. The application should run with minimum privileges. The solution for this was to create a console application (of course in .net core 3.1) that will install the certificate when run with the right privileges. The console program will have almost the same code as shown above. Here is how the docker file looked like: 

```dockerfile
FROM mcr.microsoft.com/dotnet/core/aspnet:3.1 AS base
EXPOSE 80

FROM base AS final
WORKDIR /app
COPY ./out /app

# Swithch to container admin user so that the root certificate can be added 
USER ContainerAdministrator

# download the certificate
ADD http://my.path.to/certificate.crt /app

# add the certificate to root store of local machine
RUN CertImporter.exe "C:\app\certificate.crt"

# switch back to the normal user
USER ContainerUser

ENTRYPOINT ["dotnet", "MyEntryPOint.dll"]
```

Information about windows container users is something that is not visible to the developers directly. I hope this helps.