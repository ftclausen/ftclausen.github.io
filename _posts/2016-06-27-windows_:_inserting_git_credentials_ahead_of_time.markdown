---
title: "Windows : Inserting Git Credentials Ahead Of Time"
date: 2016-06-27 14:06
classes: wide
categories: infra
---

**Problem**: Git for Windows hangs on Jenkins when attempting to use the "`store`" credential
helper.

**Solution**: Pre-populate the Windows Credential Manager with the required
credentials

## Introduction

The latest [Git for Windows](https://git-for-windows.github.io/) uses the
Windows Credential Manager which means the "store" Git credential helper is not
available. We were using the "store" credential manager so that Jenkins knew
what the Git credentials were when cloning repositories. 

This meant that Git clone operations were just hanging since there was a
user name and password box popping up.

The fix involves

* Install the GCM DLLs and prep them for use
* Write a powershell script to interface with those DLLs
* Run said script to insert credentials into the Windows Credential Manager

I'm in the process of automating this via a Chef cookbook but I'd prefer to just
explain the essence of it here so that you can adapt it to any automation you
may be using.

## Install GCM

### Download and Extract

I ended up installing the GCM Zip file alongside the regular GCM in Program
Files as I could not get the DLLs in Program Files to work. So download the
[latest Zip
file](https://github.com/Microsoft/Git-Credential-Manager-for-Windows/releases/latest)
and extract it somewhere. For this example I put it in `c:\gcm-v1.4.0`. 

### Unblock the DLLs

You need to unblock the DLLs by right clicking on them, selecting properties and
clicking "Unblock" (near the bottom).

## PowerShell Script

I used a variation of the following script to load the credentials into the
Windows Credential Manager via GCM

```
# insert_jenkins_credentials.ps1
Add-Type -Path 'c:\path\to\gcm-v1.4.0\Microsoft.Alm.Authentication.dll'

$credentials = New-Object -TypeName Microsoft.Alm.Authentication.Credential('someuser', 'secret')
$targetUri = New-Object -TypeName Microsoft.Alm.Authentication.TargetUri('https://git.example.com/projects')
$secretStore = New-Object -TypeName Microsoft.Alm.Authentication.SecretStore("git", $null, $null, $null)

$foundCredentials = $null
$secretStore.ReadCredentials($targetUri, [ref] $foundCredentials)
if ($foundCredentials -ne $null) {
    echo "Credentials already found, not inserting"
} else {
    echo "Inserting stored credentials"
    $secretStore.WriteCredentials($targetUri, $credentials)
}
```

If you're not sure of the namespace then manually allow Git for Windows to
Launch the GCM, input the credentials, and check what is saved.

### Run PowerShell Script

The above script needs to be run to ByPass various security checks due to a) the
GCM DLLs are unsigned and the script itself is unsigned. That is less than ideal
and I've asked the GCM project if they can sign their DLLs and I'll look into
signing the script itself.

#### Run as Current User

So the command line to actually execute the script is (replacing the path with
where you chose to save the script)

    powershell -NoLogo -NonInteractive -NoProfile -ExecutionPolicy Bypass -InputFormat None -File c:\temp\insert_jenkins_credentials.ps1

You can then open the Windows Credential Manager to check the credentials have
been added

#### Run as SYSTEM Account

If you're running a service as Local SYSTEM then the above command will insert
the credentials into the credentials store as whatever you run that command as
*instead* of the SYSTEM account meaning the creds will not be available to the
service. For example, a Jenkins slave agent might need some credentials for
cloning Git repos.

To work around that we can use the
(PsExec.exe)[https://technet.microsoft.com/en-us/sysinternals/psexec.aspx]
command in the following way to execute the above Powershell invocation

    psexec.exe -i -s powershell -NoLogo -NonInteractive -NoProfile -ExecutionPolicy Bypass -InputFormat None -File c:\temp\insert_jenkins_credentials.ps1

The above will insert the credentials for the SYSTEM account.

## References

These posts were very helpful in finding out how to load and use an external
library from within PowerShell

* http://stackoverflow.com/questions/3079346/how-to-reference-net-assemblies-using-powershell
* http://stackoverflow.com/questions/12923074/how-to-load-assemblies-in-powershell
* http://stackoverflow.com/questions/821744/how-to-call-a-method-with-output-parameters-in-powershell

