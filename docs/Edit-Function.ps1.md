---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5518
Published Date: 2014-10-14t23
Archived Date: 2014-10-17t18
---

# edit-function - 

## Description

opens a function in a text editor….

## Comments



## Usage



## TODO



## function

`edit-function`

## Code

`#
 #
     [CmdletBinding()]
     param(
         [Parameter(ValueFromPipelineByPropertyName=$true, Mandatory=$true)]
         $Name,
         $Editor = $global:PSEditor.Command,
         $Parameters = $global:PSEditor.Parameters
     )
     process {
 
         $file = [IO.Path]::GetTempFileName() | Rename-Item -NewName { [IO.Path]::ChangeExtension($_,".tmp.ps1") } -PassThru
         if(Test-Path Function:\$Name) {
             Set-Content $file (Get-Content Function:\$Name)
         }
 
         if(!$Editor) {
             if(Get-Command Git) { 
                 $Editor, $Parameters = (git config core.editor) -replace '^([''"]?)(.*?)\1(\s+(.*?))?$',"`$2$([char]31)`$3" -split ([char]31) | % { "$_".Trim() } | ? { $_ }
             }
             if(!$Editor -or !(Get-Command $Editor)) {
                 $Editor = Get-ChildItem C:\Program*\* -recurse -filter "sublime_text.exe" -ErrorAction SilentlyContinue | Select-Object -First 1
                 $Parameters = "-n -w"
             }
 
             if(!(Get-Command $Editor)) {
                 $Editor = "notepad"
                 $Parameters = ""
             }
 
             $global:PSEditor.Command = "$Editor"
             $global:PSEditor.Parameters = "$Parameters"
         }
 
         Start-Process $Editor $Parameters, $file -Wait
 
         Set-Content Function:\$Name ([scriptblock]::create((Get-Content $file)))
         Remove-Item $File
     }
`

