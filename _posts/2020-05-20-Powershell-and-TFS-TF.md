---
title: TF checkout and PowerShell 
tags: [PowerShell, TFS, TF, Already checked out]
style: border 
color: light 
description: A note on TF.exe and the error I was seeing while checking out from PowerShell.
---

I was working on something where we needed to modify a file using PowerShell and check in the change changes back to TFS. Checking out the fine was causing some troubles. Let's look at the details. 

The script had 'ErrorActionPreference ="Stop"'. Some of those files were checked out by other users. So, calling 'tf checkout abc.txt' was causing the script to stop. This was because tf.exe was writing to error stream and returning 0 as error code (0 means everything is alright). Just because the tf.exe was writing to the error stream further execution stopped in PowerShell. 

In this case, we needed to take more control over the tf.exe process, return value from process, error stream, and stdout stream. 

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