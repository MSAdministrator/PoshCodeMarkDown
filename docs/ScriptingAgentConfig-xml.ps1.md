---
Author: jan egil ring
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 2308
Published Date: 2011-10-17t08
Archived Date: 2016-08-30t08
---

# scriptingagentconfig.xml - 

## Description

sample scriptingagentconfig.xml for working with the exchange server 2010 scripting agent cmdlet extension agent.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 <?xml version="1.0" encoding="UTF-8" ?>
 <Configuration version="1.0">
 
 <Feature Name="MailboxProvisioningDatabase" Cmdlets="new-mailbox">
   <ApiCall Name="ProvisionDefaultProperties">
 
 switch -regex (($provisioningHandler.UserSpecifiedParameters["Name"]).substring(0,1)) 
     { 
        "[A-F]" {$databasename = "MDB A-F"} 
        "[G-L]" {$databasename = "MDB G-L"} 
        "[M-R]" {$databasename = "MDB M-R"} 
        "[S-X]" {$databasename = "MDB S-X" } 
        "[Y-Z]" {$databasename = "MDB Y-Z" } 
         default {$databasename = "MDB Y-Z" }
     }
 
 
 $user = new-object -type Microsoft.Exchange.Data.Directory.Recipient.ADUser;
 
 $user.database = "CN=$databasename,CN=Databases,CN=Exchange Administrative Group (FYDIBOHF23SPDLT),CN=Administrative Groups,CN="Exchange organization name",CN=Microsoft Exchange,CN=Services,CN=Configuration,DC=domain,DC=local";
 
 new-object -type Microsoft.Exchange.Data.Directory.Management.Mailbox -argumentlist $user
 
   </ApiCall>
  </Feature>
 
 </Configuration>
`

