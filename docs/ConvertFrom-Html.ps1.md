---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4850
Published Date: 2014-01-29t16
Archived Date: 2017-05-22t02
---

# convertfrom-html - 

## Description

a simplistic way to parse an html table into objects

## Comments



## Usage



## TODO



## function

`convertfrom-html`

## Code

`#
 #
 function ConvertFrom-Html {
    #.Synopsis
    #.Example
    param(
       [Parameter(ValueFromPipeline=$true)]
       [string]$html,
 
       [string]$TypeName
    )
    begin { $content = "$html" }
    process { $content += "$html" }
    end {
       [xml]$table = $content -replace '(?s).*<table[^>]*>(.*)</table>.*','<table>$1</table>'
 
       $header = $table.table.tr[0]  
       $data = $table.table.tr[1..1e3]
 
       foreach($row in $data){ 
          $item = @{}
 
          $h = "th"
          if(!$header.th) {
             $h = "td"
          }
          for($i=0; $i -lt $header.($h).Count; $i++){
             if($header.($h)[$i] -is [string]) {
                $item.($header.($h)[$i]) = $row.td[$i]
             } else {
                $item.($header.($h)[$i].InnerText) = $row.td[$i]
             }
          }
          Write-Verbose ($item | Out-String)
          $object = New-Object PSCustomObject -Property $item 
          if($TypeName) {
             $Object.PSTypeNames.Insert(0,$TypeName)
          }
          Write-Output $Object
       }
    }
 }
`

