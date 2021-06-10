---
title: Updating nuget version in tens of projects
tags: [powershell, TFS, NuGet, Automation]
style: border 
color: light 
description: I had to scan tens of csproj files and make the version number consistent for a NuGet. Instead of going thru the projects manually, I created a PowerShell script to correct the NuGet version.
---

The continuous integration build failed with an error. The error read "Could not load file or assembly 'Moq, Version=4.13.0.0, Culture=neutral, PublicKeyToken=69f491c39445e920' or one of its dependencies.". Reason? Someone checked in a project with a higher Moq version in a large solution with hundreds of projects. Now, someone had to go thru all of the projects to understand where was the wrong version added. 

Because this is a repeating problem, instead of going thru all the projects manually, I created a tiny script to correct the version and check out the files (I don't want to check-in things without personally looking at them).

Here is the script for your reference. I hope this helps you. You can also re-purpose this script to upgrade nuget version in a lot of projects.

```powershell

$projectpath = "C:\Projects\myfancyprojectname"
$TfExePath = "C:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\Common7\IDE\CommonExtensions\Microsoft\TeamFoundation\Team Explorer\tf.exe"
if(! (Test-Path $TfExePath)) # if path not found then try to see if local dev path works
{
    $TfExePath = "C:\Program Files (x86)\Microsoft Visual Studio\2019\Professional\Common7\IDE\CommonExtensions\Microsoft\TeamFoundation\Team Explorer\tf.exe" # uncomment this to try this on dev env.
}
if(! (Test-Path $TfExePath)) # if path not found then try to see if local dev path works
{
    $TfExePath = "C:\Program Files (x86)\Microsoft Visual Studio\2019\Enterprise\Common7\IDE\CommonExtensions\Microsoft\TeamFoundation\Team Explorer\tf.exe" # uncomment this to try this on dev env.
}

if(! (Test-Path $TfExePath))
{
    throw "TF path is not valid ($TfExePath)"
}

Set-Alias -Name tf -Value  $TfExePath


function CheckoutFiles
{
    param(
        [string] $TfExePath,
        [string] $FilesToBeChckedOut
    )
    process
    {
        Write-Host ""
        Write-Host "    More than 10 files have beeb batched for check-out. Hence, checking them out now."
        Write-Host "    Checking out files $FilesToBeChckedOut"
            
        #start of checkout
        # if a user has checked out the file on other machine then we can continue to ignore those errors.
        # Only non 0 return value should be considered as an error.
        $pinfo = New-Object System.Diagnostics.ProcessStartInfo
        $pinfo.FileName = $TfExePath
        $pinfo.RedirectStandardError = $true
        $pinfo.RedirectStandardOutput = $true
        $pinfo.UseShellExecute = $false
        #$pinfo.WorkingDirectory = $SourceDirectory
        $CheckoutCommand = " checkout   $FilesToBeChckedOut"
        $pinfo.Arguments =  $CheckoutCommand
        $p = New-Object System.Diagnostics.Process
        $p.StartInfo = $pinfo
        $p.Start() | Out-Null
        $p.WaitForExit()
        $stdout = $p.StandardOutput.ReadToEnd()
        $stderr = $p.StandardError.ReadToEnd()
        $ExitCode = $p.ExitCode
        
        if($ExitCode -ne 0)
        {
            Write-Host "exit code: $ExitCode" 
            Write-Host "CheckoutCommand is $CheckoutCommand"
            Write-Host "stdout: $stdout"
            Write-Host "stderr: $stderr"
            throw "An error occurred while checking out. Please look at stdout and stderr."
        }
    }
}

$MoqVersion = "4.16.1"
$IncorrectMoqVersionOutput =""
$FilesToBeChckedOut = ""
$Count = 0 

foreach($proj in $PackagesConfigList)
{
    $XMLfileName = $proj.FullName
    # using this strange way of loading XML document to preserve white space (more info here: https://stackoverflow.com/questions/8160613/powershell-saving-xml-and-preserving-format )
    $ProjFileXML = New-Object xml
    $ProjFileXML.PreserveWhitespace = $true

    # Load with preserve setting
    $ProjFileXML.Load($XMLfileName)
    
    $Nodes =$ProjFileXML.GetElementsByTagName("PackageReference") 
    $ProjectUpdated = $false;
    if($Nodes)
    {
        foreach($node in $Nodes)
        {   
            if( $node.Include -eq "Moq" -and $node.Version -ne $MoqVersion)
            {
                $ProjectUpdated = $true
                if($IncorrectMoqVersionOutput -eq "" )
                {
                   $IncorrectMoqVersionOutput += "        $XMLfileName (" + $node.Version  + ")";
                }
                else
                {
                    $IncorrectMoqVersionOutput += "
        $XMLfileName (" + $node.Version  + ")";
                }
                $node.Version =  $MoqVersion;
            }

           
            if($ProjectUpdated -eq $true)
            {
                Set-ItemProperty $proj -name IsReadOnly -value $false # remove read only attribute
                $ProjFileXML.Save($XMLfileName)
                $Count = $Count +1 
            
                if($Count -gt 10 )
                {
                    CheckoutFiles $TfExePath $FilesToBeChckedOut 
                    #end of checkout
                    $FilesToBeChckedOut = ""
                    $Count = 0 
                }
      
                Write-Host ""
                Write-Host "    Will check out file $proj"
                $FilesToBeChckedOut = $FilesToBeChckedOut + " ""$proj"" "
                Write-Host ""
                  
            }
        }
    }
 
    $ProjFileXML = $null
}


if($FilesToBeChckedOut -ne "")
{
    CheckoutFiles $TfExePath $FilesToBeChckedOut 
    $FilesToBeChckedOut = ""
    $Count = 0 
}

if($IncorrectMoqVersionOutput -ne "" )
{
    Write-Output "Incorrect MOQ version found in some projects (expected is $MoqVersion): "
    Write-Output $IncorrectMoqVersionOutput
}
```

Happy coding!
