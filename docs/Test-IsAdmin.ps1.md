---
Author: boe prox
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2667
Published Date: 2011-05-09t19
Archived Date: 2011-05-12t06
---

# test-isadmin - 

## Description

this advanced function will look to see if the current user context running a command/script is an administrator or not. if not, a menu is presented to the user to either continue or enter alternate credentials. the function will either return a credential object of the alternate credential or a string type stating that the current user context will be used.

## Comments



## Usage



## TODO



## function

`test-isadmin`

## Code

`#
 #
 Function Test-IsAdmin   
 {  
 .SYNOPSIS     
    Function used to detect if current user is an Administrator.  
      
 .DESCRIPTION   
    Function used to detect if current user is an Administrator. Presents a menu if not an Administrator  
       
 .NOTES     
     Name: Test-IsAdmin  
     Author: Boe Prox   
     DateCreated: 30April2011    
       
 .EXAMPLE     
     Test-IsAdmin  
       
    
 Description   
 -----------       
 Command will check the current user to see if an Administrator. If not, a menu is presented to the user to either  
 continue as the current user context or enter alternate credentials to use. If alternate credentials are used, then  
 the [System.Management.Automation.PSCredential] object is returned by the function.  
 #>  
     [cmdletbinding()]  
     Param()  
       
     Write-Verbose "Checking to see if current user context is Administrator"  
     If (-NOT ([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole] "Administrator"))  
     {  
         Write-Warning "You are not currently running this under an Administrator account! `nThere is potential that this command could fail if not running under an Administrator account."  
         Write-Verbose "Presenting option for user to pick whether to continue as current user or use alternate credentials"  
         $choice = [System.Management.Automation.Host.ChoiceDescription[]] @("Use &Alternate Credentials","&Continue with current Credentials")  
   
         [int]$default = 0  
   
         $userchoice = $host.ui.PromptforChoice("Warning","Please select to use Alternate Credentials or current credentials to run command",$choice,$default)  
   
         Write-Debug "Selection: $userchoice"  
   
         Switch ($Userchoice)  
         {  
             0  
             {  
                 Write-Verbose "Prompting for Alternate Credentials"  
                 $Credential = Get-Credential  
                 Write-Output $Credential      
             }  
             1  
             {  
                 Write-Verbose "Using current credentials"  
                 Write-Output "CurrentUser"  
             }  
         }          
           
     }  
     Else   
     {  
         Write-Verbose "Passed Administrator check"  
     }  
 }
`

