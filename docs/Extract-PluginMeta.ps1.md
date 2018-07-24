---
Author: mario
Publisher: 
Copyright: 
Email: 
Version: 0.3
Encoding: utf-8
License: public domain
PoshCode ID: 6832
Published Date: 2017-04-04t18
Archived Date: 2017-04-07t08
---

# pmd extraction - 

## Description

provides a crude implementation of plugin meta data extraction

## Comments



## Usage



## TODO



## functions

`extract-pluginmeta`

## Code

`#
 )
 #
 #
 #
 #
 
 
 #-- baseline plugin meta data support
 function Extract-PluginMeta() {
     <#
       .SYNOPSIS
          Extract plugin meta data from given filename
       .DESCRIPTION
          Reads specified file, returns hashtable for key:value entries
          from topmost comment block. Prepares config{} dicts.
       .PARAMETER fn
          Script to read from
       .OUTPUTS
       .EXAMPLE
          For instance to read a couple of "plugins" at once:
            $menu = (Get-Item tools/*.ps1 | % { Extract-PluginMeta $_ })
          Which builds a usable hashtable list from asorted scripts,
          once they have been documented. Which thus eases processing:
            Show-MenuCLI ($menu | % { $_.title -and $_.type -eq "inline" })
          Or executing "init" plugins for instance:
            $menu | ? { $_.api -eq "mytool" -and $_.type -eq "init" } | { . $_.fn }
       .TEST
          (Extract-PluginMeta $MyInvocation.MyCommand.Definition).version -eq "0.3"
       .NOTES
          Packs comment remainder as `doc` field.
     #>
     Param($fn, $meta=@{}, $cfg=@())
 
     $str = Get-Content $fn | Out-String
 
     if ($m = [regex]::match($str, '(?m)((?:^\s*[#]+.*$\n?)+)')) {
 
         $str = $m.groups[1] -replace "(?m)^\s*#[ \t]*", ""
         $str, $doc = [regex]::split($str, '(?m)^$\n')
 
         preg_match_all -rx "(?m)^([\w-]+):\s*(.*(?:[\r\n]*^\s+.*$)*)" -str $str | % { $meta[$_[1]] = $_[2].trim() }
 
         preg_match_all -rx "\{(.+?)\}" -str $meta.config | % { $r = @{};
             preg_match_all -rx "([\w]+)\s*[:=]\s*(?:[']?([^,;}]+)[']?)" -str $_[1] | % {  $r[$_[1]] = $_[2] }; $cfg += $r;
         }
 
         $meta.fn = $fn
         $meta.doc = ($doc -join "`r`n")
         $meta.config = $cfg
     }
 }
 
 #-- Regex/Select-String -Allmatches as nested array / a convenience shortcut /  well, why, yes; it's a PHP-style function
 function preg_match_all() {
     Param($rx, $str)
     $str | Select-String $rx -AllMatches | % { $_.Matches } | % { ,($_.Groups | % { $_.Value }) }
 }
`

