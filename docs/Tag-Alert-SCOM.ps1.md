---
Author: stefan stranger
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 635
Published Date: 
Archived Date: 2008-10-13t20
---

# tag-alert (scom) - 

## Description

tags alert with principalname, severity and mp name in custom fields.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $alerts = get-alert | where {$_.principalname -ne $null -and $_.resolutionstate -eq "0"}
 foreach($alert in $alerts)
       { 
             $alert.CustomField1 = $alert.PrincipalName 
             $alert.CustomField2 = $alert.Severity 
             if ($alert.IsMonitorAlert -like 'False') 
                   { 
                         $alert.CustomField3 = ((get-rule $alert.monitoringruleid).getmanagementpack()).displayname 
                   } 
             else 
                   { 
                         $alert.CustomField3 = ((get-monitor $alert.problemid).getmanagementpack()).displayname 
                   } 
                   $alert.Update("") 
       }
`

