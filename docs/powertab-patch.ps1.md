---
Author: idvorkin
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1586
Published Date: 
Archived Date: 2010-01-22t07
---

# powertab patch - 

## Description

patch to tabexpansion.ps1

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
     filter convertTypeToTabCompletionName()
     {
 
         $typeToTabCompletion=@{
         [Microsoft.Powershell.Commands.X509StoreLocation]={$_.Location};
         [System.Security.Cryptography.X509Certificates.X509Store]={$_.Name};
         [Microsoft.Win32.RegistryKey]={ $_.Name.Split("\")[-1] };
         }
 
         $convertToTabCompletionFunction =$typeToTabCompletion[$_.GetType()]
         if (-not $convertToTabCompletionFunction  )
         {
             $convertToTabCompletionFunction  = {$_.Name}
         }
 
         $_ | % $convertToTabCompletionFunction 
     }
 
       $ChildItems | convertTypeToTabCompletionName |% {$container + "\" + $_}| Invoke-TabItemSelector $lastPath -SelectionHandler $SelectionHandler -return $lastword -ForceList |% {
`

