---
title: TF checkout and PowerShell 
tags: [PowerShell, TFS, TF, Already checked out]
style: border 
color: light 
description: A note on TF.exe and the error I was seeing while checking out from PowerShell.
---

Calling "tf checkout" writes to the error stream if the file is checked out by any other user in any workspace. Powershell by default considers any error stream output as an error. Hence, checking out a file using TF from the PowerShell did not work well (The script had 'ErrorActionPreference ="Stop"').   

Even though output was written to error stream by tf.exe, 0 was returned from the process (0 means everything is alright). Just because the tf.exe was writing to the error stream, PowerShell did not execute further code. It was necessary to continue further execution and ignore the error stream if the return value was 0. 

I took more control over the tf.exe process with help from "System.Diagnostics.Process". With this, we can get return value from process, error stream, and stdout stream as a variable. 

Here is the code change: 

```PowerShell

$FileToBeCheckedOut = "abc.txt" # file path that needs to be checked out.
$TfExePath = "C:\Program Files (x86)\Microsoft Visual Studio\2017\Professional\Common7\IDE\CommonExtensions\Microsoft\TeamFoundation\Team Explorer\tf.exe"

Set-Alias -Name tf -Value  $TfExePath

# if a user has checked out the file on other machine then we can continue to ignore those errors.
# Only non 0 return value should be considered as an error.
$pinfo = New-Object System.Diagnostics.ProcessStartInfo
$pinfo.FileName = $TfExePath
$pinfo.RedirectStandardError = $true
$pinfo.RedirectStandardOutput = $true
$pinfo.UseShellExecute = $false
#$pinfo.WorkingDirectory = $SourceDirectory
$CheckoutCommand = " checkout   $FileToBeCheckedOut"
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

```
