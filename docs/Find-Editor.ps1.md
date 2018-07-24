---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5529
Published Date: 2014-10-21t01
Archived Date: 2014-12-24t21
---

# find-editor - 

## Description

a tool to find a text editor

## Comments



## Usage



## TODO



## function

`find-editor`

## Code

`#
 #
 
         function Find-Editor {
             #.Synopsis
             #.Description
             [CmdletBinding()]
             param
             (
                 [Parameter(Position=1)] 
                 [System.String]
                 $Editor = $(if($global:PSEditor.Command){ $global:PSEditor.Command } else { $global:PSEditor } ),
 
                 [Parameter(Position=2)]
                 [System.String]
                 $Parameters = $global:PSEditor.Parameters
             )
             begin {
                 function Split-Command {
                     param(
                         [Parameter(Mandatory=$true)]
                         [string]$Command
                     )
                     $Parts = @($Command -Split " ")
 
                     for($count=$Parts.Length; $count -gt 1; $count--) {
                         $Editor = ($Parts[0..$count] -join " ").Trim("'",'"')
                         if((Test-Path $Editor) -and (Get-Command $Editor -ErrorAction SilentlyContinue)) { 
                             $Editor
                             $Parts[$($Count+1)..$($Parts.Length)] -join " "
                             break
                         }
                     }
                 }
             }
 
             end {
                     if($Editor -and (Get-Command $Editor)) { break }
 
                     if ($Editor -and !(Get-Command $Editor -ErrorAction SilentlyContinue))
                     {
                         Write-Verbose "Editor is not a valid command, split it:"
                         $Editor, $Parameters = Split-Command $Editor
                         if($Editor) { break }
                     }
 
                     if (Test-Path Env:Editor)
                     {
                         Write-Verbose "Editor was not passed in, trying Env:Editor"
                         $Editor, $Parameters = Split-Command $Env:Editor
                         if($Editor) { break }
                     }
 
                     if (Get-Command Git -ErrorAction SilentlyContinue)
                     {
                         Write-Verbose "PSEditor and Env:Editor not found, searching git config"
                         $Editor, $Parameters = Split-Command (git config core.editor)
                         if($Editor) { break }
                     }
 
                     Write-Verbose "Editor not found, trying some others"
                     if($Editor = Get-Command "subl.exe", "sublime_text.exe" | Select-Object -Expand Path -first 1)
                     {
                         $Parameters = "-n -w"
                         break
                     }
                     if($Editor = Get-Command "notepad++.exe" | Select-Object -Expand Path -first 1)
                     {
                         $Parameters = "-multiInst"
                         break
                     }
                     if($Editor = Get-Command "atom.exe" | Select-Object -Expand Path -first 1)
                     {
                         break
                     }
 
                     Write-Verbose "Editor still not found, getting desperate:"
                     if(($Editor = Get-Item "C:\Program Files\DevTools\Sublime Text ?\sublime_text.exe" -ErrorAction SilentlyContinue | Sort {$_.VersionInfo.FileVersion} -Descending | Select-Object -First 1) -or
                        ($Editor = Get-ChildItem C:\Program*\* -recurse -filter "sublime_text.exe" -ErrorAction SilentlyContinue | Select-Object -First 1))
                     {
                         $Parameters = "-n -w"
                         break
                     }
 
                     if(($Editor = Get-ChildItem "C:\Program Files\Notepad++\notepad++.exe" -recurse -filter "notepad++.exe" -ErrorAction SilentlyContinue | Select-Object -First 1) -or
                        ($Editor = Get-ChildItem C:\Program*\* -recurse -filter "notepad++.exe" -ErrorAction SilentlyContinue | Select-Object -First 1))
                     {
                         $Parameters = "-multiInst"
                         break
                     }
 
                     Write-Verbose "Editor not found, settling for notepad"
                     $Editor = "notepad"
 
                     if(!$Editor -or !(Get-Command $Editor -ErrorAction SilentlyContinue -ErrorVariable NotFound)) { 
                         if($NotFound) { $PSCmdlet.ThrowTerminatingError( $NotFound[0] ) }
                         else { 
                             throw "Could not find an editor (not even notepad!)"
                         }
                     }
                 } while($false)
 
                 $PSEditor = New-Object PSObject -Property @{
                         Command = "$Editor"
                         Parameters = "$Parameters"
                 } | Add-Member ScriptMethod ToString -Value { "'" + $this.Command + "' " + $this.Parameters } -Force -PassThru
 
                 if($PSBoundParameters.ContainsKey("Editor") -or 
                    $PSBoundParameters.ContainsKey("Parameters") -or 
                    !(Test-Path variable:global:PSeditor) -or 
                    ($PSEditor.Command -ne $Editor))
                 {
                     Write-Verbose "Setting global preference variable for Editor: PSEditor"
                     $global:PSEditor = $PSEditor
 
                     if(![Environment]::GetEnvironmentVariable("Editor", "User")) {
                         Write-Verbose "Setting user environment variable: Editor"
                         [Environment]::SetEnvironmentVariable("Editor", "$PSEditor", "User")
                     }
                 }
                 return $PSEditor
             }
         }
`

