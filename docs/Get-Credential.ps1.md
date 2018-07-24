---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.2
Encoding: ascii
License: cc0
PoshCode ID: 681
Published Date: 
Archived Date: 2009-01-05t16
---

# get-credential - 

## Description

an improvement over the default get-credential cmdlet (doesn’t take much, eh?) offering, among other things, a -console switch.

## Comments



## Usage



## TODO



## function

`get-credential`

## Code

`#
 #
 ###################################################################################################
 ###################################################################################################
 function Get-Credential{ 
 PARAM([String]$Title, [String]$Message, [String]$Domain, [String]$UserName, [switch]$Console)
    $ShellIdKey = "HKLM:\SOFTWARE\Microsoft\PowerShell\1\ShellIds"
    $cp = (Get-ItemProperty $ShellIdKey ConsolePrompting -ea "SilentlyContinue")
    $cp = $cp.ConsolePrompting -eq $True
 
    if($Console -and !$cp) {
       Set-ItemProperty $ShellIdKey ConsolePrompting $True
    } elseif(!$Console -and $Console.IsPresent -and $cp) {
       Set-ItemProperty $ShellIdKey ConsolePrompting $False
    }
 
    $Host.UI.PromptForCredential($Title,$Message,$UserName,$Domain)
 
    Set-ItemProperty $ShellIdKey ConsolePrompting $cp
 }
`

