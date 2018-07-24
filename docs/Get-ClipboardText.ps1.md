---
Author: matthew sessions
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4980
Published Date: 2015-03-12t21
Archived Date: 2015-01-31t20
---

# get-clipboardtext - 

## Description

this function retrieves the (unicode) text from the clipboard.

## Comments



## Usage



## TODO



## function

`get-clipboardtext`

## Code

`#
 #
 Function Get-ClipboardText
 {
     [CmdletBinding()]
     [OutputType([String])]
     
     
     [System.Windows.Forms.Clipboard]::GetText( 'UnicodeText' ) | Write-Output
 }
`

