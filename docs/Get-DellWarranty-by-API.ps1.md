---
Author: dane kantner
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6363
Published Date: 2016-05-31t10
Archived Date: 2016-06-02t13
---

# get-dellwarranty by api - 

## Description

get-dellwarranty (uses new dell api; the old get-dellwarranty scripts screen scrape their site and no longer work because they changed the formatting of the tables. this relies on their api service so in theory it should be maintained by dell and remain working. the script itself has an array of computer names at the top and will cycle through each system and query that warranty information. based off of all warranty lines, the highest warranty is tracked and outputted as well.  the get-dellwarranty function itself is configured to accept a -servicetag or -serialnumber parameter or accept piped input, and outputs an object that contains objects of warranty entitlement lines.

## Comments



## Usage



## TODO



## script

`get-dellwarranty`

## Code

`#
 #
 ##
 
 $computers="localhost","Chiv5908-2009","anyothercomputers","NYSPC-JJAJ68YG6"
 
 foreach ($computer in $computers) {
 
 $obj=get-wmiobject win32_systemenclosure -computername $computer -ErrorAction SilentlyContinue
 
         write-output "Computer $computer unavailable (offline or WMI inaccessible).`n"
     } else {
         $WarrantyObject=Get-DellWarranty -ServiceTag $obj.SerialNumber | select @{name = "ComputerName";expression = {$Computer}},ServiceLevelCode,ServiceLevelDescription,Provider,StartDate,EndDate,DaysLeft,EntitlementType   
 
         $DaysLeft=0
         $HighestServiceDesc=""
         foreach ($line in $WarrantyObject) {
                 $DaysLeft=$line.DaysLeft
                 $HighestServiceDesc=$line.ServiceLevelDescription
                 }
             write-output $line
             }
         
         if ($warrantyObject -ne $null) { 
             write-output "Maximum warranty for computer $Computer has $DaysLeft days remaining. $HighestServiceDesc`n"
         } else {
             write-output "Dell returned no warranty information for $Computer. Is it a Dell?"   
        
       
 
 
 Function Get-DellWarranty {
     [CmdletBinding()]
         param(
             [Parameter(Mandatory=$False,Position=0,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
             [alias("serialnumber")]
             [string[]]$ServiceTag
         )
     $WebProxy=New-WebServiceProxy -Uri http://xserv.dell.com/services/assetservice.asmx
     $WarrantyInformation=$WebProxy.GetAssetInformation(([guid]::NewGuid()).Guid,"Dell warranty",$serviceTag)
     $WarrantyInformation | Select-Object -ExpandProperty Entitlements
     return $WarrantyInformation
 }
`

