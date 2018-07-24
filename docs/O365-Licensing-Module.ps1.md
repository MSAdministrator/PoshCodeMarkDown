---
Author: boardwithlife
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6153
Published Date: 2016-12-22t17
Archived Date: 2016-03-19t02
---

# o365 licensing module - 

## Description

this is a module i create to make getting office 365 account info a bit easier.

## Comments



## Usage



## TODO



## function

`set-maillicense`

## Code

`#
 #
 Function Set-MailLicense {
 <#
 .Synopsis
    Tool for Office 365 Mailbox Licensing
 .DESCRIPTION
    This function is used to add and/or remove COMPANY Licenses for users. The -UserName parameter will accept multiple usernames
    passed through the pipeline. Set-MailLicense will also accept a "UserName" property sent through the pipeline. When licensing a user for the first time, the location parameter is mandatory. 
 .EXAMPLE
    Set-MailLicense -Username Test.User -Add -Location US
    <This will add the default Enterprise License to a users account and set their UsageLocation to US. To see a full
    list of all available locations, type the -Location Parameter at the prompt and hit space to see the drop down list.>
 .EXAMPLE
    Set-MailLicense -Username Test.User -Remove
    <This will remove the COMPANY license from the desired user.>
 .Notes
     
 #>
     [CmdletBinding()]
     Param(
         [Parameter(Mandatory=$true,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true,Position=0)]
         [string[]]$UserName,
         [Parameter(ParameterSetName='Remove')]
         [switch]$Remove,
         [Parameter(ParameterSetName='Add')]
         [switch]$Add,
         [Parameter(Mandatory=$True,ParameterSetName='add')]
         [validateset('US','CA','CN','GB','HK','FI','AE')]
         [string]$Location
     )
     Begin {
         $OldEA = $ErrorActionPreference
         $ErrorActionPreference = "SilentlyContinue"
     }
     Process {
             If($Remove){
                 Set-MsolUserLicense -UserPrincipalName "$UserName@domain.com" -RemoveLicenses "COMPANY365:ENTERPRISEPACK"
             }
                 Else{
                     Set-MsolUser -UserPrincipalName "$UserName@domain.com" -UsageLocation $Location 
                     Set-MsolUserLicense -UserPrincipalName "$UserName@domain.com" -AddLicenses "COMPANY365:ENTERPRISEPACK"
                 }
             Get-MailLicense -UserName $UserName
     }
     End{
         $ErrorActionPreference = $OldEA
     }
 }
 
 Function Get-MailLicense {
 <#
 .Synopsis
    This Function retrieves a COMPANY users mailbox status
 .DESCRIPTION
    This function is used to find a COMPANY users mailbox license status and type. 
 .EXAMPLE
    PS C:\Scripts> Get-MailLicense test.user
 
 
     UserName    : Test.User
     Office      : San Francisco
     WhenCreated : 8/12/2014 19:53:35
     Licenses    : {COMPANY365:ENTERPRISEPACK}
     IsLicensed  : True
 
    <This shows the Office 365 account info provided by a small selection of the Get-MSOLUser Cmdlet output.>
 .EXAMPLE
    Set-MailLicense -Username Test.User -Remove
    <This will remove the HOK license from the desired user.>
 #>
     [CmdletBinding()]
     Param (
         [Parameter(Mandatory=$true,Position=0)]
         [string[]]$UserName
     )
     Process {
         Get-MsolUser -UserPrincipalName "$UserName@doamin.com" | Select @{E={$_.DisplayName.Replace(" ",".")};L='UserName'},Office,WhenCreated,Licenses,isLicensed
     }
 }
`

