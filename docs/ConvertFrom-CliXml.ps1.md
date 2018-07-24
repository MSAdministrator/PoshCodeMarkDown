---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5743
Published Date: 2016-02-19t12
Archived Date: 2016-03-20t09
---

# convertfrom-clixml - 

## Description

a pair with convertto-clixml — this version closes and disposes the string reader handle. now works in powershell 3 and 4 as well.

## Comments



## Usage



## TODO



## function

`convertfrom-clixml`

## Code

`#
 #
 function ConvertFrom-CliXml {
     param(
         [Parameter(Position=0, Mandatory=$true, ValueFromPipeline=$true)]
         [ValidateNotNullOrEmpty()]
         [String[]]$InputObject
     )
     begin
     {
         $OFS = "`n"
         [String]$xmlString = ""
     }
     process
     {
         $xmlString += $InputObject
     }
     end
     {
         $type = [PSObject].Assembly.GetType('System.Management.Automation.Deserializer')
         $ctor = $type.GetConstructor('instance,nonpublic', $null, @([xml.xmlreader]), $null)
         $sr = New-Object System.IO.StringReader $xmlString
         $xr = New-Object System.Xml.XmlTextReader $sr
         $deserializer = $ctor.Invoke($xr)
         $done = $type.GetMethod('Done', [System.Reflection.BindingFlags]'nonpublic,instance')
         while (!$type.InvokeMember("Done", "InvokeMethod,NonPublic,Instance", $null, $deserializer, @()))
         {
             try {
                 $type.InvokeMember("Deserialize", "InvokeMethod,NonPublic,Instance", $null, $deserializer, @())
             } catch {
                 Write-Warning "Could not deserialize ${string}: $_"
             }
         }
         $xr.Close()
         $sr.Dispose()
     }
 }
`

