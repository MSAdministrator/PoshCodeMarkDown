---
Author: poshoholic
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4544
Published Date: 2016-10-23t01
Archived Date: 2016-03-05t18
---

# convertto-clixml - 

## Description

export-clixml and import-clixml only work with files. this is an implementation for sending clixml directly to the pipeline. requires powershell 2+ (now works in 3.0 and 4.0)

## Comments



## Usage



## TODO



## function

`convertto-clixml`

## Code

`#
 #
 function ConvertTo-CliXml {
     param(
         [Parameter(Position=0, Mandatory=$true, ValueFromPipeline=$true)]
         [ValidateNotNullOrEmpty()]
         [PSObject[]]$InputObject
     )
     begin {
         $type = [PSObject].Assembly.GetType('System.Management.Automation.Serializer')
         $ctor = $type.GetConstructor('instance,nonpublic', $null, @([System.Xml.XmlWriter]), $null)
         $sw = New-Object System.IO.StringWriter
         $xw = New-Object System.Xml.XmlTextWriter $sw
         $serializer = $ctor.Invoke($xw)
         #$method = $type.GetMethod('Serialize', 'nonpublic,instance', $null, [type[]]@([object]), $null)
     }
     process {
         try {
             [void]$type.InvokeMember("Serialize", "InvokeMethod,NonPublic,Instance", $null, $serializer, [object[]]@($InputObject))
         } catch {
             Write-Warning "Could not serialize $($InputObject.GetType()): $_"
         }
     }
     end {    
         [void]$type.InvokeMember("Done", "InvokeMethod,NonPublic,Instance", $null, $serializer, @())
         $sw.ToString()
         $xw.Close()
         $sw.Dispose()
     }
 }
`

