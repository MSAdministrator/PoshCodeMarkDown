---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 679
Published Date: 
Archived Date: 2009-01-05t17
---

# set-rdpsetting - 

## Description

function/script to set settings in a rdp file for terminal services.  supports pipeline input and smart conversion of bools.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ########################################################################################################################
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 
     param(
         [string]$Path = $(throw "A path to a RDP file is required."),
         [string]$Name = "",
         [Object]$Value = "",
         [switch]$Passthru
     )
     
     begin {
         if (Test-Path $path) {
                $content = Get-Content -Path $path
         } else {
             throw "Path does not exist."
         }
         
         $datatypes = @{
             'allow desktop composition' = 'i';
             'allow font smoothing' = 'i';
             'alternate shell' = 's';
             'audiomode' = 'i';
             'authentication level' = 'i';
             'auto connect' = 'i';
             'autoreconnection enabled' = 'i';
             'bitmapcachepersistenable' = 'i';
             'compression' = 'i';
             'connect to console' = 'i';
             'desktopheight' = 'i';
             'desktopwidth' = 'i';
             'disable cursor setting' = 'i';
             'disable full window drag' = 'i';
             'disable menu anims' = 'i';
             'disable themes' = 'i';
             'disable wallpaper' = 'i';
             'displayconnectionbar' = 'i';
             'domain' = 's';
             'drivestoredirect' = 's';
             'full address' = 's';
             'gatewaycredentialssource' = 'i';
             'gatewayhostname' = 's';
             'gatewayprofileusagemethod' = 'i';
             'gatewayusagemethod' = 'i';
             'keyboardhook' = 'i';
             'maximizeshell' = 'i';
             'negotiate security layer' = 'i';
             'password 51' = 'b';
             'prompt for credentials' = 'i';
             'promptcredentialonce' = 'i';
             'port' = 'i';
             'redirectclipboard' = 'i';
             'redirectcomports' = 'i';
             'redirectdrives' = 'i';
             'redirectposdevices' = 'i';
             'redirectprinters' = 'i';
             'redirectsmartcards' = 'i';
             'remoteapplicationmode' = 'i';
             'screen mode id' = 'i';
             'server port' = 'i';
             'session bpp' = 'i';
             'shell working directory' = 's';
             'smart sizing' = 'i';
             'username' = 's';
             'winposstr' = 's'
         }
     }
     
     process {
         if ($_.Name) {
             $tempname = $_.Name
             $tempvalue = $_.Value
             if ($datatypes[$tempname] -eq 'i') {
                 if (($tempvalue -eq $true) -or ($tempvalue -imatch '^true$')) {
                     $tempvalue = 1
                 } elseif (($tempvalue -eq $false) -or ($tempvalue -imatch '^false$') -or ($tempvalue -eq '')) {
                     $tempvalue = 0
                 } elseif ($tempvalue -match '^[0-9]+$') {
                     $tempvalue = [int]$tempvalue
                 }
             } else {
                 $tempvalue = [string]$tempvalue
             }
             
             $found = $false
             $content = $content | ForEach-Object {
                 if ($_ -match "^$tempname") {
                     "${tempname}:$($datatypes[$tempname]):$tempvalue"
                     $found = $true
                 } else {$_}
             }
             if (!$found) {
                 $content += @("${tempname}:$($datatypes[$tempname]):$tempvalue")
             }
         }
     }
     
     end {
         if ($name) {
             if ($datatypes[$name] -eq 'i') {
                 if (($value -eq $true) -or ($value -imatch '^true$')) {
                     [int]$value = 1
                 } elseif (($value -eq $false) -or ($value -imatch '^false$') -or ($value -eq '')) {
                     [int]$value = 0
                 } elseif ($value -match '^[0-9]+$') {
                     [int]$value = $value
                 }
             } else {
                 [string]$value = $value
             }
             
             $found = $false
             $content = $content | ForEach-Object {
                 if ($_ -match "^$name") {
                     "${name}:$($datatypes[$name]):$value"
                     $found = $true
                 } else {$_}
             }
             if (!$found) {
                 $content += @("${name}:$($datatypes[$name]):$value")
             }
         }
         
         $content | Set-Content -Path $path
         if ($passthru) {
             Get-ChildItem -Path $path
         }
     }
 #}
`

