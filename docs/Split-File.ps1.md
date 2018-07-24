---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5522
Published Date: 2015-10-17t19
Archived Date: 2015-01-31t20
---

# split-file - 

## Description

splits a file into many files based on a regular expression

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 #.Synopsis
 #.Example
 #
 #.Example
 #
 
 [CmdletBinding()]
 param(
     [Parameter(Mandatory=$true, Position=0)]
     [string]$Path,
 
     [Text.Encoding]$Encoding = [Text.Encoding]::Default,
 
     [Parameter(ParameterSetName="Pattern", Mandatory=$true, Position=0)]
     [string]$Pattern,
 
     [String]$Header
 
 )
 $Path = Convert-Path $Path
 Write-Verbose "Opening Reader for $Path"
 $Reader = New-Object IO.StreamReader $Path, $Encoding
 $Extension = [IO.Path]::GetExtension($Path)
 
 $FileCount = 0
 $LineIndex = 0
 try {
     if($Pattern) {
         sls -Path $Path -pattern $Pattern | ForEach {
             $Match = $_
 
             $FileCount += 1
             $FileName = [IO.Path]::ChangeExtension($Path, ".${FileCount}${Extension}")
             $Writer = New-Object IO.StreamWriter $FileName, $false, $Encoding
 
             Write-Verbose "Writing $($Match.LineNumber - $LineIndex) lines to $(Resolve-Path $FileName -Relative)"
             if($Header) { $Writer.Write($Header + "`r`n") }
             try {
                 for(; $LineIndex -lt $Match.LineNumber; $LineIndex++) {
                     $Writer.Write( $Reader.ReadLine() + "`r`n")
                 }
             } catch {
                 throw $_
             } finally {
                 Write-Debug "Closing Writer"
                 $Writer.Close()
             }
         }
 
         $FileCount += 1
         $FileName = [IO.Path]::ChangeExtension($Path, ".${FileCount}${Extension}")
         $Writer = New-Object IO.StreamWriter $FileName, $false, $Encoding
         Write-Verbose "Writing the rest to $(Resolve-Path $FileName -Relative)"
         $LastFile = $LineIndex
         try {
             while($Reader.Peek() -ge 0) {
                 $LineIndex++
                 $Writer.Write( $Reader.ReadLine() + "`r`n")
             }
         } catch {
             throw $_
         } finally {
             Write-Verbose "Wrote $($LineIndex -$LastFile) lines to $(Resolve-Path $FileName -Relative)"
             Write-Debug "Closing Writer"
             $Writer.Close()
         }
     }
 } finally {
     Write-Debug "Closing Reader"
     $Reader.Close()
 }
`

