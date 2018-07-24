---
Author: mario
Publisher: 
Copyright: 
Email: 
Version: 0.9
Encoding: ascii
License: pd
PoshCode ID: 6830
Published Date: 2017-04-04t18
Archived Date: 2017-04-08t04
---

# wpf + winforms - 

## Description

winform and wpf shortcuts

## Comments



## Usage



## TODO



## functions

``

## Code

`#
 #
 #
 #
 #
 #
 #
 #
 
 
 #-- register libs
 Add-Type -AN PresentationCore, PresentationFramework, WindowsBase
 Add-Type -AN System.Drawing, System.Windows.Forms, Microsoft.VisualBasic
 
 
 #-- WPF/WinForms widget wrapper
 function W {
     [CmdletBinding()]
     Param($type = "Button", $prop = @{}, $Base="Controls")
 
     if ($type.getType().Name -eq "String") {
         $w = New-Object System.Windows.$Base.$type
     }
     else {
         $w = $type
     }
 
     $prop.keys | % {
         $key = $_
         $val = $prop[$_]
         if ($pt = ($w | Get-Member -Force -Name $key)) { $pt = $pt.MemberType }
 
         if ($pt -eq "Property") {
             if (($Base -eq "Forms") -and (@("Size" , "ItemSize", "Location") -contains $key)) {
                 $val = New-Object System.Drawing.Size($val[0], $val[1])
             }
             $w.$key = $val
         }
         else {
             ForEach ($obj in @($w, $w.Children, $w.Child, $w.Container)) {
                 if ($obj.psobject -and $obj.psobject.methods.match($key) -and $obj.$key) {
                     ([array]$val) | % { $obj.$key.Invoke($_) } | Out-Null
                     break
                 }
             }
         }
     }
     return $w
 }
 
 #-- WinForms version
 function WF {
     Param($type, $prop, $add=@{}, $click=$null)
     W -Base Forms -Type $type -Prop ($prop + @{add=$add; add_click=$click})
 }
 
 #-- Document "widgets"
 function WD {
     Param($type, $prop=@{})
     W -Base Documents -Type $type -Prop $prop
 }
`

