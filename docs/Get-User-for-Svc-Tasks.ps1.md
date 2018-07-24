---
Author: stephen wheet
Publisher: 
Copyright: 
Email: 
Version: 2.0
Encoding: ascii
License: cc0
PoshCode ID: 2082
Published Date: 2013-08-17t16
Archived Date: 2013-05-19t11
---

# get user for svc/tasks - getserviceandtaskaccounts.ps1

## Description

comment

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #==========================================================================
 #
 #
 #
 #
 #==========================================================================
 
 get-pssnapin -registered | add-pssnapin -passthru
 $ReqVersion = [version]"1.2.2.1254" 
 
 if($QadVersion -lt $ReqVersion) { 
     throw "Quest AD cmdlets version '$ReqVersion' is required. Please download the latest version" 
 
 $ErrorActionPreference = "SilentlyContinue"
 
 $IgnoreAcct = "NT AUTHORITY\LocalService",
       "LocalSystem",
       ".\*", 
       "NT AUTHORITY\NETWORK SERVICE", 
       "NT AUTHORITY\NetworkService"
 
 $list = "Server,Service,Account,"
 $list | format-table | Out-File "c:\reports\GetSVCAccts\SVCAccounts.csv"
 $list2 = "Server,Task,Next Run Time,Account,"
 $list2 | format-table | Out-File "c:\reports\GetSVCAccts\TASKAccounts.csv"
 
 
 
 Foreach ($server in $Servers ) {
     $serverFQDN = $server.dnsname
 	$ping = new-object System.Net.NetworkInformation.Ping
 	$Reply = $ping.Send($serverFQDN)
     if ($Reply.status -eq "Success"){
         Write-Host "$serverFQDN ************* Online"
 		
         $error.clear()
 		gwmi win32_service -computer $serverFQDN -property name, startname, caption |
 			% {
                 $name = $_.Name
                 $Acctname = $_.StartName
                 If ( $IgnoreAcct -notcontains $AcctName )
                 { 
                     Write-host "$serverFQDN   $Name   $Acctname"
                     $list = "$serverFQDN,$Name,$Acctname"
     			    $list | format-table | Out-File -append "d:\reports\GetSVCAccts\SVCAccounts.csv"
         
 			if (!$?) {
                 $errmsg = "$serverFQDN,No RPC server,ACCESS DENIED"
                 $errmsg | format-table | Out-File -append "d:\reports\GetSVCAccts\SVCAccounts.csv"
             
        $SchQuery = Schtasks.exe /query /s $serverFQDN /NH /V /FO CSV  
             If ($SchQuery -ne "INFO: There are no scheduled tasks present in the system.") 
             {
                 ForEach ($Sch in $SchQuery)
                 {
                     Write-host "*********************" 
                     $Schfixed = $Sch.Replace("`"","")
                     $Props = $Schfixed.Split(',')
                 
                     ForEach ($Prop in $Props)
                     {
                         If ($Prop -like "firm\*")
                         {
                             $list2 = $Props[0],$Props[1],$Props[2],$Prop
                             $list2 | format-table | Out-File -append "d:\reports\GetSVCAccts\TASKAccounts.csv"
                             Write-host $list
             Else 
             {
                 $list2 = $serverFQDN,$SchQuery
                 $list2 | format-table | Out-File -append "d:\reports\GetSVCAccts\TASKAccounts.csv"
                 Write-host $list
     Else
     {
         Write-Host "$serverFQDN ************* OffLine"
         $list = "$serverFQDN,OFFLINE"
         $list | format-table | Out-File -append "d:\reports\GetSVCAccts\SVCAccounts.csv"
`

