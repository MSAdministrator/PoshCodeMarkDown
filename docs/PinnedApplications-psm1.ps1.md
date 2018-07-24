---
Author: jan egil ring
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 1667
Published Date: 2010-02-26t07
Archived Date: 2017-04-28t02
---

# pinnedapplications.psm1 - 

## Description

powershell-module with the ability to pin and unpin programs from the taskbar and the start-menu in windows 7 and windows server 2008 r2.

## Comments



## Usage



## TODO



## module

`set-pinnedapplication`

## Code

`#
 #
  #
  
  #
  #
  #
  
  function Set-PinnedApplication
   {
 .SYNOPSIS 
 This function are used to pin and unpin programs from the taskbar and Start-menu in Windows 7 and Windows Server 2008 R2
 .DESCRIPTION 
 The function have to parameteres which are mandatory:
 Action: PinToTaskbar, PinToStartMenu, UnPinFromTaskbar, UnPinFromStartMenu
 FilePath: The path to the program to perform the action on
 .EXAMPLE
 Set-PinnedApplication -Action PinToTaskbar -FilePath "C:\WINDOWS\system32\notepad.exe"
 .EXAMPLE
 Set-PinnedApplication -Action UnPinFromTaskbar -FilePath "C:\WINDOWS\system32\notepad.exe"
 .EXAMPLE
 Set-PinnedApplication -Action PinToStartMenu -FilePath "C:\WINDOWS\system32\notepad.exe"
 .EXAMPLE
 Set-PinnedApplication -Action UnPinFromStartMenu -FilePath "C:\WINDOWS\system32\notepad.exe"
 #> 
    [CmdletBinding()]
    param(
       [Parameter(Mandatory=$true)][string]$Action, [string]$FilePath
    )
    
   
  switch ($Action) {
  
  "PinToTaskbar" {
   $PinVerbTask = @{}
  $PinverbTask['En-US'] ="Pin to Taskbar" 
  $PinverbTask['Nb-No'] ="Fest til oppgavelinjen"
 
  
    $culture = $Host.CurrentUICulture.Name
    
        if(-not (test-path $FilePath)) { write-warning "`nPath does not exist.`n $FilePath `nExiting... `n";  return  }
           
        $path= split-path $FilePath
        $shell=new-object -com "Shell.Application" 
        $folder=$shell.Namespace($path)   
        $item = $folder.Parsename((split-path $FilePath -leaf))
       
        $verbs=$item.Verbs()    
 	   
 	   foreach($v in $verbs){if($v.Name.Replace("&","") -match $PinVerbTask[$culture]){$v.DoIt()}} 
 
  }
  "PinToStartMenu" {
   $PinVerbStart = @{}
   $PinverbStart['En-US'] ="Pin to Start Menu" 
   $PinverbStart['Nb-No'] ="Fest til Start-menyen" 
   
    $culture = $Host.CurrentUICulture.Name
  
        if(-not (test-path $FilePath)) { write-warning "`nPath does not exist.`n $FilePath `nExiting... `n";  return  }
           
        $path= split-path $FilePath
        $shell=new-object -com "Shell.Application" 
        $folder=$shell.Namespace($path)   
        $item = $folder.Parsename((split-path $FilePath -leaf))
       
        $verbs=$item.Verbs()    
 	   
 	   foreach($v in $verbs){if($v.Name.Replace("&","") -match $PinVerbStart[$culture]){$v.DoIt()}}
  
  }
  "UnPinFromTaskbar" {
    $UnPinVerbTask = @{}
  $UnPinverbTask['En-US'] ="Unpin from Taskbar" 
  $UnPinverbTask['Nb-No'] ="Løsne programmet fra oppgavelinjen"
 
    $culture = $Host.CurrentUICulture.Name
 
               if(-not (test-path $FilePath)) { write-warning "`nPath does not exist.`n $FilePath `nExiting... `n";  return  }
           
        $path= split-path $FilePath
        $shell=new-object -com "Shell.Application" 
        $folder=$shell.Namespace($path)   
        $item = $folder.Parsename((split-path $FilePath -leaf))
       
        $verbs=$item.Verbs()    
  
  foreach($v in $verbs){if($v.Name.Replace("&","") -match $UnPinVerbTask[$culture]){$v.DoIt()}}
  }
  "UnPinFromStartMenu" {
    $UnPinVerbStart = @{}
   $UnPinverbStart['En-US'] ="Unpin from Start Menu" 
   $UnPinverbStart['Nb-No'] ="Løsne fra Start-menyen"
  
   
    $culture = $Host.CurrentUICulture.Name
 
               if(-not (test-path $FilePath)) { write-warning "`nPath does not exist.`n $FilePath `nExiting... `n";  return  }
           
        $path= split-path $FilePath
        $shell=new-object -com "Shell.Application" 
        $folder=$shell.Namespace($path)   
        $item = $folder.Parsename((split-path $FilePath -leaf))
       
        $verbs=$item.Verbs()    
  foreach($v in $verbs){if($v.Name.Replace("&","") -match $UnPinVerbStart[$culture]){$v.DoIt()}}
  }
  default {Write-Warning "Invalid action specified. Valid actions are: PinToTaskbar, PinToStartMenu, UnPinFromTaskbar, UnPinFromStartMenu"}
  }
  }
`

