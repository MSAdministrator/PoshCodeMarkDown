---
Author: oisin grehan
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3712
Published Date: 2015-10-24t11
Archived Date: 2015-05-06t21
---

# get-helpinfo - ps v3 - 

## Description

shows information about locally installed help files for your modules, alongside the available version online should you run update-help. this is for powershell 3.0 only. with thanks to shay and alexandair for useful snippets.

## Comments



## Usage



## TODO



## function

`get-helpinfo`

## Code

`#
 #
 
 function Get-HelpInfo {
 
     write-progress -id 1 -Activity "Get-HelpInfo" -Status "Getting remote version table..."
 
     $remote = & {
         $html = invoke-webrequest http://technet.microsoft.com/en-us/windowsserver/jj662920
         $html.ParsedHtml.getElementsByTagName("TABLE").item(2).rows| % -begin {
             $total = $html.ParsedHtml.getElementsByTagName("TABLE").item(2).rows.length
             $index = 0
         } -process {
             write-progress -id 1 -Activity "Get-HelpInfo" -Status "Getting remote version table..." `
                 -PercentComplete (([float]$index / $total) * 100)
 		    ($_.cells | % innertext) -join ","
             $index++
         } | convertfrom-csv
 	}
 
     write-progress -id 1 -Activity "Get-HelpInfo" -Status "Enumerating local modules..." `
         -CurrentOperation "Please wait..."
 
     gmo -list | ? {
         write-progress -id 1 -Activity "Get-HelpInfo" -Status "Examining $($_.name) ..."
         test-path ($info = join-path (split-path $_.path) `
             ("{0}_{1}_HelpInfo.xml" -f $_.name, $_.guid))
     } | % {
         $name = $_.Name
         ([xml](gc $info)).HelpInfo.SupportedUICultures.UICulture
     } | % {
         [pscustomobject]@{
             ModuleName = $name
             UICulture = $_.UICultureName
             Installed = $_.UICultureVersion
             Available = ($remote|? { ($_.modulename -eq $name) -and $_.uiculture `
                 -contains $_.uiculturename } ).version
         }
     }
 }
 
 get-helpinfo | ft -AutoSize
`

