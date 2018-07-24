---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2640
Published Date: 2012-04-27t23
Archived Date: 2015-01-03t07
---

# convertto-pseudotype - 

## Description

converts objects to custom psobjects with robust type support. allows converting data from import-csv etc into type-safe pseudo structs …

## Comments



## Usage



## TODO



## function

`convertto-pseudotype`

## Code

`#
 #
 
 
 <#
 .Synopsis
    Convert an object to a custom PSObject wiith robust type information
 .Parameter TypeName
    The name(s) of the PseudoType(s) to be inserted into the objects for the sake of formatting
 .Parameter Mapping
    A Hashtable of property names to types (or conversion scripts)
 .Parameter InputObject
    An object to convert.
 .Example
    Get-ChildItem | Where { !$_.PsIsContainer } | Export-CSV files.csv
    $Mapping = @{ 
       Attributes         = [System.IO.FileAttributes]
       CreationTime       = [System.DateTime]
       CreationTimeUtc    = [System.DateTime]
       Directory          = [System.IO.DirectoryInfo]
       DirectoryName      = [System.String]
       Exists             = [System.Boolean]
       Extension          = [System.String]
       FullName           = [System.String]
       IsReadOnly         = [System.Boolean]
       LastAccessTime     = [System.DateTime]
       LastAccessTimeUtc  = [System.DateTime]
       LastWriteTime      = [System.DateTime]
       LastWriteTimeUtc   = [System.DateTime]
       Length             = [System.Int64]
       Name               = [System.String]
       PSChildName        = [System.String]
       PSDrive            = [System.Management.Automation.PSDriveInfo]
       PSIsContainer      = [System.Boolean]
       PSParentPath       = [System.String]
       PSPath             = [System.String]
       PSProvider         = { Get-PSProvider $_ }
       ReparsePoint       = [System.Management.Automation.PSCustomObject]
       VersionInfo        = [System.Diagnostics.FileVersionInfo]
    }
    
    Import-CSV | ConvertTo-PseudoType Selected.System.IO.FileInfo, System.IO.FileInfo $Mapping
    
    Get-ChildItem | Where { !$_.PsIsContainer } | Select *
    
    NOTE: Not all types are rehydrateable from CSV output -- the "VersionInfo" will be hydrated as a string...
 #>
 [CmdletBinding()]
 param(
    [Parameter(Mandatory=$true, Position=0)]
    [Alias("Name","Tn")]
    [String[]]$TypeName
 ,
    [Parameter(Mandatory=$true, Position=1)]
    [Hashtable]$Mapping
 ,
    [Parameter(Mandatory=$true, Position=99, ValueFromPipeline=$true)]
    [PSObject[]]$InputObject
 )
 begin {
    $MappingFunction = @{}
    foreach($key in $($Mapping.Keys)) {
       $MappingFunction.$Key = {$_.$Key -as $Mapping.$Key}
    }
    [Array]::Reverse($TypeName)
 }
 process {
    foreach($IO in $InputObject) {
       $Properties = @{}
       foreach($key in $($Mapping.Keys)) {
          if($Mapping.$Key -is [ScriptBlock]) {
             $Properties.$Key = $IO.$Key | ForEach-Object $Mapping.$Key
          } elseif($Mapping.$Key -is [Type]) {
             if($Value = $IO.$Key -as $Mapping.$Key) {
                $Properties.$Key = $Value
             } else {
                $Properties.$Key = $IO.$Key
             }
          } else {
             $Properties.$Key = [PSObject]$IO.$Key
          }
       }
       New-Object PSObject -Property $Properties | %{ foreach($type in $TypeName) { $_.PSTypeNames.Insert(0, $type) } $_ }
     }
 }
 
 
 #}
`

