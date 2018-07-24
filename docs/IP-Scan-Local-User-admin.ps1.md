---
Author: niklas
Publisher: 
Copyright: 
Email: 
Version: 10.1.1
Encoding: ascii
License: cc0
PoshCode ID: 1684
Published Date: 2012-03-05t07
Archived Date: 2012-01-12t06
---

# ip scan/local user admin - 

## Description

where i work, we don’t use ad for roughly 30-60 servers. there are multiple identical local windows accounts on various servers, and when a person leaves the company, those accounts need to be deleted by hand. this group of scripts performs the following tasks

## Comments



## Usage



## TODO



## class

`addremove-localaccount`

## Code

`#
 #
 $obj = New-Object system.Net.NetworkInformation.Ping
 100..200 | % { $ip = "10.1.1.$_"
 $ping = $obj.send($ip,100)
 write-host "." -noNewLine
 if ($ping.status -eq "Success"){
    C:\Powershell\Finduser.ps1 $ping.address.ipaddresstostring
 }}
 
 #--------------------------------
 
 param (
         [string]$strcomputer,
         [switch]$delete)
 
   $computer = [ADSI]("WinNT://" + $strComputer + ",computer")
  
   $Users = $computer.psbase.children |where{$_.psbase.schemaclassname -eq "User"}
   foreach ($member in $Users.psbase.syncroot){
     if ($member.name -eq "username"){
       write-host Found user! $computer.name
       if ($delete){
         write-host Deleting...
         .\set-localaccount.ps1 -UserName username -remove -computerName $computer.name
       }
     }
   }
 
 #--------------------------------
 
 ##################################################################################
 #
 #
 ##################################################################################
 
 param([string]$UserName, [string]$Password, [switch]$Add, [switch]$Remove, [switch]$ResetPassword, [switch]$help, [string]$computername)
 
 function GetHelp() {
 $HelpText = @"
 DESCRIPTION:
 
 NAME: Set-LocalAccount.ps1
 Adds or Removes a Local Account
 
 PARAMETERS:
 
 -UserName        Name of the User to Add or Remove (Required)
 -Password        Sets Users Password (optional)
 -Add             Adds Local User (Optional)
 -Remove          Removes Local User (Optional)
 -ResetPassword   Resets Local User Password (Optional)
 -help            Prints the HelpFile (Optional)
 
 SYNTAX:
 
 .\Set-LocalAccount.ps1 -UserName nika -Password Password1 -Add
 Adds Local User nika and sets Password to Password1
 
 .\Set-LocalAccount.ps1 -UserName nika -Remove
 Removes Local User nika
 
 .\Set-LocalAccount.ps1 -UserName nika -Password Password1 -ResetPassword
 Sets Local User nika's Password to Password1
 
 .\Set-LocalAdmin.ps1 -help
 Displays the helptext
 "@
 $HelpText
 }
 
 function AddRemove-LocalAccount ([string]$UserName, [string]$Password, [switch]$Add, [switch]$Remove, [switch]$ResetPassword, [string]$computerName) {
     if($Add) {
         [string]$ConnectionString = "WinNT://$computerName"
         $ADSI = [adsi]$ConnectionString
         $User = $ADSI.Create("user",$UserName)
         $User.SetPassword($Password)
         $User.SetInfo()
     }
 
     if($Remove) {
         [string]$ConnectionString = "WinNT://$computerName"
         $ADSI = [adsi]$ConnectionString
         $ADSI.Delete("user",$UserName)
     }
 
     if($ResetPassword) {
         [string]$ConnectionString = "WinNT://" + $ComputerName + "/" + $UserName + ",user"
         $Account = [adsi]$ConnectionString
         $Account.psbase.invoke("SetPassword", $Password)
     }
 }
 
 if($help) { GetHelp; Continue }
 if($UserName -AND $Password -AND $Add -AND !$ResetPassword) { AddRemove-LocalAccount -UserName $UserName -Password $Password -Add -computerName $computerName}
 if($UserName -AND $Password -AND $ResetPassword) { AddRemove-LocalAccount -UserName $UserName -Password $Password -ResetPassword -computerName $computerName}
 if($UserName -AND $Remove) { AddRemove-LocalAccount -UserName $UserName -Remove -computerName $computerName}
`

