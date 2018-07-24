---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.1
Encoding: ascii
License: cc0
PoshCode ID: 3748
Published Date: 2012-11-06t07
Archived Date: 2012-11-10t18
---

# inifile functions - 

## Description

a bunch of functions for working with .ini files… (updated to gracefully handle ini files with only one section by detecting if $sections is an array of matches or just a single match)

## Comments



## Usage



## TODO



## function

`get-inisection`

## Code

`#
 #
 function Get-IniSection($inifile,$section)
 {
    $sections = select-string "^\[.*\]" $inifile
    if(!$section) {
       return $sections | %{$_.Line.Trim("[]")}
    }
    $start = 0
    switch($sections){
       {$_.Line.Trim() -eq "[$section]"}{
          $start = $_.LineNumber -1
          
       }
       default {
          if($start){ 
             return (gc $inifile)[($start)..($start + ($_.LineNumber-2 - $start))]
          }
       }
    }
    $lines = gc $inifile
    return $lines[$start..($lines.length-1)]
 }
 
 
 function Get-IniValue($inifile,$section,$name)
 {
    $section = Get-IniSection $inifile $section
    ($section | Select-String "^\s*$name\s*=").Line.Split("=",2)[1]
 }
 
 function Set-IniValue($inifile,$section,$name,$value)
 {
    $lines = gc $inifile
    $sections = select-string "^\[.*\]" $inifile
    $start,$end = 0,0
    if($sections.GetType().IsArray) {
       for($l=0; $l -lt $sections.Count; ++$l){
          if($sections[$l].Line.Trim() -eq "[$section]") {
             $start = $sections[$l].LineNumber
             if($l+1 -ge $sections.Count) {
                $end = $lines.length-1;
             } else {
                $end = $sections[$l+1].LineNumber -2
             }
          }
       }
    } else {
       if($sections.Line.Trim() -eq "[$section]") {
          $start = $sections.LineNumber
          $end = $lines.length-1
       }
    }
    
    if($start -and $end) {
       $done = $false
       for($l=$start;$l -le $end;++$l){
          if( $lines[$l] -match "^\s*$name\s*=" ) {
             $lines[$l] = "{0} = {1}" -f $name, $value
             $done = $true
             break;
          }
       }
       if(!$done) {
          $output = $lines[0..$start]
          $output += "{0} = {1}" -f $name, $value
          $output += $lines[($start+1)..($lines.Length-1)]
          $lines = $output
       }
    }
    
    Set-Content $inifile $lines
 }
 
 
 
 ##
 function Get-IniFile {
 param([string]$inifile=$(Throw "You must specify the name of an ini file!"))
    $INI = @{}
    $s,$k,$v = $null
    foreach($line in (gc $inifile | ? {$_[0] -ne ";" -and $_.Trim().Length -gt 0})) {
       $k,$v = $line.Split("=",2)
       if($v -eq $null) {
          $s = $k.Trim("[]")
          $INI[$s] = @{}
       } else {
          $INI[$s][$k.Trim()] = $v.Trim()
       }
    }
    return $INI
 }
 
 function Set-IniFile {
 param([HashTable]$ini,[string]$inifile=$(Throw "You must specify the name of an ini file!"))
    [string[]]$inistring = @()
    foreach($section in $ini.Keys) {
       $inistring += ("`n[{0}]" -f $section)
       foreach($key in $ini[$section].Keys) {
          $inistring += ("{0} = {1}" -f $key, $ini[$section][$key])
       }
    }
    Set-Content $inifile $inistring
 }
`

