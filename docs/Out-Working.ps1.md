---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 105
Published Date: 
Archived Date: 2008-10-05t09
---

# out-working - 

## Description

display a “working” animation without knowing how much work you’ll be doing …

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ##
 ##
 ##
 ##
 
 param($fore="White",$back="red")
 $work = @( $Host.UI.RawUI.NewBufferCellArray(@("|"),$fore,$back),
            $Host.UI.RawUI.NewBufferCellArray(@("/"),$fore,$back),
            $Host.UI.RawUI.NewBufferCellArray(@("-"),$fore,$back),
            $Host.UI.RawUI.NewBufferCellArray(@("\"),$fore,$back) );
 
 [int]$script:w = 0;
 
 filter out-working($wait=0) {
    $cur = $Host.UI.RawUI.Get_CursorPosition(); 
    $cur.X = 0; $cur.Y -=1;
    $Host.UI.RawUI.SetBufferContents($cur,$work[$script:w++]);
    if($script:w -gt 3) {$script:w = 0 }
    Start-Sleep -milli $wait
    $_
 }
`

