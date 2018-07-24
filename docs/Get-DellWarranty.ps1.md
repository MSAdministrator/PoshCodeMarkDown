---
Author: jasser
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 3648
Published Date: 2012-09-17t07
Archived Date: 2012-09-21t03
---

# get-dellwarranty - 

## Description

queries the dell website for a list of service tags and returns the warranty information as a custom object.

## Comments



## Usage



## TODO



## function

`get-dellwarranty`

## Code

`#
 #
 function Get-DellWarranty {
     <#
     .Synopsis
         Provides warranty information for one or more Dell service tags.
     .Description
         Queries the Dell Website for a list of service tags and returns the warranty information as a custom object.
         If a service tag has multiple warranties, they are returned as multiple separate objects.
         If no service tag is specified, it will attempt to retrieve and lookup the local system's service tag.
         
         The service tag can be provided to the command in several ways: 
         1. Using the -servicetag parameter
         2. By passing one or more Service Tags via the pipeline
         3. By passing objects that have either servicetag or serialnumber defined as a property, such as win32_bios WMI objects
          
         See examples for details.
     .Parameter ServiceTag
         ALIAS: serialnumber
         The Dell service tag you wish to query. Example: XYZ12A3
     .Example
         C:\PS> Get-DellWarranty
         
         Service Tag                 : XXXX123
         Description                 : 4 Hour On-Site Service
         Provider                    : UNY
         Warranty Extension Notice * : No
         Start Date                  : 6/19/2009
         End Date                    : 6/20/2011
         Days Left                   : 140
         
         Description
         -----------
         If no service tags are provided, the script retrieves the local computer's serial number from WMI and queries for it.
     .Example
         C:\PS> Get-DellWarranty -ServiceTag XXXX123
         
         Service Tag                 : XXXX123
         Description                 : 4 Hour On-Site Service
         Provider                    : UNY
         Warranty Extension Notice * : No
         Start Date                  : 6/19/2009
         End Date                    : 6/20/2011
         Days Left                   : 140
         
         Description
         -----------
         You can pass the service tag as a parameter, or as the first positional parameter (e.g. Get-DellWarranty XXXX123)
     
     .Example
         C:\PS> "XXXX123","XXXX124","XXXX125" | get-dellwarranty
         Service Tag                 : XXXX123
         Description                 : 4 Hour On-Site Service
         Provider                    : UNY
         Warranty Extension Notice * : No
         Start Date                  : 6/19/2009
         End Date                    : 6/20/2011
         Days Left                   : 140
         
         Service Tag                 : XXXX124
         Description                 : 4 Hour On-Site Service
         Provider                    : UNY
         Warranty Extension Notice * : No
         Start Date                  : 6/14/2009
         End Date                    : 6/15/2011
         Days Left                   : 145
         
         Service Tag                 : XXXX125
         Description                 : NBD On-Site Service
         Provider                    : DELL
         Warranty Extension Notice * : No
         Start Date                  : 6/14/2008
         End Date                    : 6/15/2010
         Days Left                   : 0
         
         Description
         -----------
         You can pass serial numbers via the pipeline either directly or as a variable.
         
     .Example
         C:\PS> get-wmiobject win32_bios -computername "computer1","computer2","1.2.3.4" | get-dellwarranty
         Service Tag                 : XXXX123
         Description                 : 4 Hour On-Site Service
         Provider                    : UNY
         Warranty Extension Notice * : No
         Start Date                  : 6/19/2009
         End Date                    : 6/20/2011
         Days Left                   : 140
         
         Service Tag                 : XXXX124
         Description                 : 4 Hour On-Site Service
         Provider                    : UNY
         Warranty Extension Notice * : No
         Start Date                  : 6/14/2009
         End Date                    : 6/15/2011
         Days Left                   : 145
         
         Service Tag                 : XXXX125
         Description                 : NBD On-Site Service
         Provider                    : DELL
         Warranty Extension Notice * : No
         Start Date                  : 6/14/2008
         End Date                    : 6/15/2010
         Days Left                   : 0
         
         Description
         -----------
         You can also pass any object that has a "serialnumber" or "servicetag" property. In this example, we query computers directly for their serial numbers,
         and pass those results (which are WMI objects that have a serialnumber property) via pipeline directly to the command to obtain warranty information.
     
     .Notes
         AUTHOR:     Justin Grote <jgrote NOSPAM-AT allieddigital NOSPAM-DOT net>
         WARNING:    Since Dell does not provide a formal API, this script works by screen-scraping the HTML from the Dell product support site. 
                     Any future change to the layout or logic of this site will break the script or cause unpredictable results.
         
         HISTORY:    v1.0 [31 Jan 2011] - Initial Module Creatio
     .Link
         http://support.dell.com/support/topics/global.aspx/support/my_systems_info
 
     #>
 
 
     [CmdletBinding()]
     param(
         [Parameter(Mandatory=$False,Position=0,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
         [alias("serialnumber")]
         [string[]]$ServiceTag
     )
 
     PROCESS {
     
         if ($ServiceTag -eq $null) {
             write-verbose "No Service Tags provided. Using Local Computer's Service Tag"
             write-verbose "START Obtaining Serial number via WMI for localhost"
             $ServiceTag = (get-wmiobject win32_bios).SerialNumber
             write-verbose "SUCCESS Obtaining Serial number via WMI for localhost - $ServiceTag"
         }
         
         foreach ($strServicetag in $servicetag) {
                 write-verbose "START Querying Dell for Service Tag $_"
                 Get-DellWarrantyWorker $strServicetag
                 write-verbose "SUCCESS Querying Dell for Service Tag $_"
         }
     }
 
 }
 
 Function Get-DellWarrantyWorker {
   Param(
     [String]$serviceTag
   )
   $URL = "http://support.dell.com/support/topics/global.aspx/support/my_systems_info/details?c=us&l=en&s=gen&ServiceTag=$serviceTag"
   
   trap [System.FormatException] { 
     write-error -category invalidresult "The service tag $serviceTag was not found. This is either because you entered the tag incorrectly, it is not present in Dell's database, or Dell changed the format of their website causing this search to fail." -recommendedaction "Please check that you entered the service tag correctly"
     return;
   }
   
   $HTML = (New-Object Net.WebClient).DownloadString($URL)
   If ($HTML -Match '<table[\w\s\d"=%]*contract_table">.+?</table>') {
     $htmltable = $Matches[0]
   } else {
     throw (New-Object System.FormatException)
   }
   $HtmlLines = $htmltable -Split "<tr" | Where-Object { $_ -Match '<td' }
   $Header = ($HtmlLines[0] -Split '<td') -Replace '[\w\s\d"=%:;\-]*>|</.*' | Where-Object { $_ -ne '' }
   
   For ($i = 1; $i -lt $HtmlLines.Count; $i++) {
     $Output = New-Object PSObject
     $Output | Add-Member NoteProperty "Service Tag" -value $serviceTag
     $Values = ($HtmlLines[$i] -Split '<td') -Replace '[\w\s\d"=%:;\-]*>|</.*|<a.+?>'
     For ($j = 1; $j -lt $Values.Count; $j++) {
       $Output | Add-Member NoteProperty $Header[$j - 1] -Value $Values[$j]
     }
     
     if ($output.'Days Left' -match '<<0') {write-host -fore darkgreen "match!";$output.'Days Left' = 0}
     
     return $Output
   }
 }
`

