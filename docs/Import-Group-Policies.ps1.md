---
Author: adam liquorish
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3109
Published Date: 2012-12-19t18
Archived Date: 2012-06-22t17
---

# import group policies - 

## Description

import bulk group policies by only specifying the import directory.  all group policies will be imported to domain.  script is currently only able to be run from server 2008 r2.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ##############################################
 ##
 ##*Currently will only run on Server 2008 R2
 ##*Script is based on the Microsoft Visual Basic Script for importing group policies.
 ##*Server doesnt need to be defined as PDC emulator will be contacted by default
 ##############################################
 
 
 param(
 
 [Parameter(Mandatory=$true,
 
     HelpMessage="Enter Path for input ie c:\Temp\output.html.")]
 
     [ValidateNotNullOrEmpty()]
 
     [string]$outputpath=$(Read-Host -prompt "Path for input"),
 
 [Parameter(Mandatory=$true,
 
     HelpMessage="Enter Domain")]
 
     [ValidateNotNullOrEmpty()]
 
     [string]$computername=$(Read-Host -prompt "Domain Name")
 
 )
 import-module grouppolicy
 
 
 
 
 $start=get-date
 
 $prog=1
 
 $i=$null
 
 $a=$null
 
 $b=$null
 
 $c=$null
 
 $max=$null
 
 
 
 function GPOName ($b)
 
 {
 
 ([xml](get-content $b)).GPO.Name
 
 }
 
 
 $a=get-childitem -recurse $path
 
 $b=$a|where-object {$_.Name -eq "gpreport.xml"}
 
 $c=foreach ($test in $b){GPOName($test.fullname)}
 
 $c|
 
 Foreach-object {
 
 
 $max=$b.length
 
 $i++
 
 
 Import-GPO -BackupGpoName $_ -TargetName $_ -path $path -CreateIfNeeded -Dom $domain
 
 
 Write-Progress -activity "Adding GPO's" -status "Added $i of $max" -PercentComplete (($i/$b.length)*100) -CurrentOperation $_}
 
 Write-Host "Completed....Total Number of GPO's imported:"$b.length
`

