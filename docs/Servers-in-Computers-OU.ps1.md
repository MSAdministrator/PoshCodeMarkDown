---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 848
Published Date: 
Archived Date: 2009-02-26t17
---

# servers in computers ou - 

## Description

many organizations have a separate ous for their servers and workstations, this is an example on how to list servers in the wrong ou.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function checkADSI()
 { $ADSIStat = "Servers in Computer OU"
   $error.clear()
   $errcnt = 0
     
   $objDomain = New-Object System.DirectoryServices.DirectoryEntry("LDAP://CN=Computers,DC=domain,DC=com")
   $objSearcher = New-Object System.DirectoryServices.DirectorySearcher($objDomain)
   $objSearcher.Filter = ("(operatingSystem=*Server*)")
   $objSearcher.SearchScope = "OneLevel"
   
   $colProplist = "dnshostname"
   foreach ($i in $colPropList)
   {$idx = $objSearcher.PropertiesToLoad.Add($i)}
 
   $colResults = $objSearcher.FindAll()
   for($i = 0; $i -lt ($colResults.count -1); $i++ ) 
   { $objResult = $colResults[$i]
     $objComputer = $objResult.Properties
     $cName = ($objComputer.dnshostname -replace ".domain.com","")
     
     $ADSIStat += "`n`t! " + $cname
     $errcnt += 1
     
   }
   if($error[0])
   { $ADSIStat += "`n`t Errors Occured: "
     foreach($err in $error)
     { $ADSIStat += "`n`t`!" + $err }
   } 
   elseif($errCnt -eq 0)
   {$ADSIStat += " None" }
   
   return $ADSIStat
`

