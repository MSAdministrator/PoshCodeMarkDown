---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 2.1
Encoding: ascii
License: cc0
PoshCode ID: 2635
Published Date: 2011-04-26t19
Archived Date: 2017-05-17t23
---

# get-application - 

## Description

attempt to resolve the path to an application using get-command or the “app paths” registry key or the start menu search.

## Comments



## Usage



## TODO



## function

`find-application`

## Code

`#
 #
 function Find-Application {
 [CmdletBinding()]
    param( [string]$Name )
 begin {
    [String[]]$Path = (Get-Content Env:Path).Split(";") |
                      ForEach-Object { 
                         if($_ -match "%.*?%"){
                            $expansion = Get-Content ($_ -replace '.*%(.*?)%.*','Env:$1')
                            ($_ -replace "%.*?%", $expansion).ToLower().Trim("\","/") + "\"
                         } else {
                            $_.ToLower().Trim("\","/") + "\"
                         }
                      }
 }
 end {
    $eap, $ErrorActionPreference = $ErrorActionPreference, "SilentlyContinue"
    $command = Get-Command $Name -Type Application
    $ErrorActionPreference = $eap
 
    if(!$command) {
       $AppPaths = "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\App Paths"
       if(Test-Path $AppPaths\$Name) {
          $default = (Get-ItemProperty $AppPaths\$Name)."(default)"
          if($default) {
          }
       }
       if(!$command) {
          $Name = [IO.Path]::GetFileNameWithoutExtension($Name)
          if(Test-Path $AppPaths\$Name) {
             $default = (Get-ItemProperty $AppPaths\$Name)."(default)"
             if($default) {
             }
          }
       }
       if(!$command){
          if(Test-Path "$AppPaths\$Name.exe") {
             $default = (Get-ItemProperty "$AppPaths\$Name.exe")."(default)"
             if($default) {
             }
          }
       }
    }
 
    if(!$command) {
       $Name = [IO.Path]::GetFileNameWithoutExtension($Name)
       $query =@"
 SELECT System.ItemName, System.Link.TargetParsingPath FROM SystemIndex
 WHERE  System.Kind = 'link' AND System.ItemPathDisplay like '%Start Menu%' AND
        ( System.Link.TargetParsingPath like '%${Name}.exe' OR System.ItemName like '%${Name}%' )
 "@
 
       $Connection = New-Object System.Data.OleDb.OleDbConnection "Provider=Search.CollatorDSO;Extended Properties='Application=Windows';"
       $Adapter = New-Object System.Data.OleDb.OleDbDataAdapter (New-Object System.Data.OleDb.OleDbCommand $Query, $Connection)
       $DataSet = New-Object System.Data.DataSet
       $Connection.Open()
       if($Adapter.Fill($DataSet)) {
          $command = $DataSet.Tables[0] | ForEach-Object { 
                      Get-Command $_["System.Link.TargetParsingPath"] | 
                      Add-Member -Type NoteProperty -Name Shortcut -Value $_["System.ItemName"] -Passthru 
                   }
       }
       $Connection.Close()
    }
 
    $( foreach($cmd in $command) {
          Get-Command $cmd.Path -ErrorAction SilentlyContinue -Type Application | ForEach-Object {
             Add-Member -InputObject $_ -Type NoteProperty -Name Shortcut -Value $cmd.ShortCut -Passthru
          }
       }
    ) | Sort-Object Definition -Unique | Sort-Object {
          $index = [array]::IndexOf( $Path, [IO.Path]::GetDirectoryName($_.Path).ToLower().Trim("\","/") + "\" )
          if($index -lt 0) { $index = 1e5 } else { $index *= 100 }
          if($_.Shortcut) {
             $index += $_.Shortcut.Length
          }
          $index
       }
 }
 
 #.Synopsis 
 #.Example
 #.Example
 #.Example
 #.Example
 #.Notes
 }
`

