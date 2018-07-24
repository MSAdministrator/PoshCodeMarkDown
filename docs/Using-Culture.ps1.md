---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4120
Published Date: 
Archived Date: 2013-04-24t09
---

#  - 

## Description

a cmdlet to run a script/scriptblock under a different culture (language). useful for testing localization.

## Comments



## Usage



## TODO



## function

`using-culture`

## Code

`#
 #
 function Using-Culture
 {
     <#
         .SYNOPSIS
             Runs a PowerShell script under a different locale to test localization features.
         .DESCRIPTION
             Runs a PowerShell script under a different locale to test localization features.
             Copied from http://rkeithhill.wordpress.com/2009/10/21/windows-powershell-2-0-string-localization/
             and converted to a Cmdlet including help comments.
         .LINK
             Import-LocalizedData
             Convert-FromStringData
             http://rkeithhill.wordpress.com/2009/10/21/windows-powershell-2-0-string-localization/
         .PARAMETER Culture
             The culture (language) to run the script in.
         .PARAMETER Script
             The scriptblock or script (wrapped in a scriptblock) to run.
         .EXAMPLE
             PS C:\> Using-Culture fr-FR { .\test.ps1 }
 
             Runs the script test.ps1 under French language settings.
     #>
         
     [CmdletBinding()]
 
     Param([Parameter(Mandatory = $true, HelpMessage = 'The culture (language) to run the script in.')]
           [ValidateNotNull()]
           [System.Globalization.CultureInfo]
           $Culture,
           [Parameter(Mandatory = $true, HelpMessage = 'The scriptblock or wrapped script to run.')]
           [ValidateNotNull()]
           [ScriptBlock]
           $Script
     )
 
     $OldCulture = [System.Threading.Thread]::CurrentThread.CurrentCulture
     $OldUICulture = [System.Threading.Thread]::CurrentThread.CurrentUICulture
     
     try
     {
         [System.Threading.Thread]::CurrentThread.CurrentCulture = $culture
         [System.Threading.Thread]::CurrentThread.CurrentUICulture = $culture
         
         Invoke-Command $script
     }
     finally
     {
         [System.Threading.Thread]::CurrentThread.CurrentCulture = $OldCulture
         [System.Threading.Thread]::CurrentThread.CurrentUICulture = $OldUICulture
     }
 }
`

