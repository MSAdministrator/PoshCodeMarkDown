---
Author: oisin grehan
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2327
Published Date: 
Archived Date: 2010-11-02t18
---

# select-enumeratedtype - 

## Description

updated 2010/10/28

## Comments



## Usage



## TODO



## script

`select-enumeratedtype`

## Code

`#
 #
 
 if (-not(test-path variable:script:_helpcache))
 {
     $SCRIPT:_helpCache = @{}
 }
 
 function Select-EnumeratedType
 {
 <#
     .SYNOPSIS
         Visually create an instance of an enum.
         
     .DESCRIPTION
         Visually create an instance of an enum with an easy to use menu system.
         Supports both single value enumerated types and bitmask (flags) enums.
         Also supports inline help for enumerated values (powershell.exe only)
     
     .PARAMETER EnumeratedType
         The enum type to provide choices for. Pass as a string or Type.
     
     .PARAMETER IncludeHelp
         Allows the caller to view help for the enumerated type, including all values.
         This help is pulled from the local .NET installation and does not require online
         access.
         
     .PARAMETER DefaultValue 
         A dynamic parameter that is added and typed to the provided enum. Use this to
         provide default values should the caller press <enter> without choosing a value
         or value(s). This parameter will absorb all trailing arguments.
         (ValueFromRemainingArguments = $true)
         
     .EXAMPLE
         $rights = Select-EnumeratedType System.Security.AccessControl.CryptoKeyRights Delete,Synchronize
         
     .EXAMPLE
         $access = Select-EnumeratedType System.IO.FileAccess Read -IncludeHelp
 #>
     [cmdletbinding()]
     param(
         [parameter(position=0, mandatory=$true)]
         [validatescript({ $_.isenum })]
         [validatenotnull()]
         [type]$EnumeratedType,
         
         [parameter()]
         [switch]$IncludeHelp
     )
     
     dynamicparam {
     
         if ($PSCmdlet.MyInvocation.BoundParameters["EnumeratedType"]) {
         
             write-verbose "Adding DefaultValue ($enumeratedType) parameter."
             
             $dict = new-object management.automation.RuntimeDefinedParameterDictionary
             $defaultParam = new-object System.Management.Automation.RuntimeDefinedParameter
             $defaultParam.Name = "DefaultValue"
             $defaultParam.ParameterType = $enumeratedType
             
             $p = new-object System.Management.Automation.ParameterAttribute
             $p.ParameterSetName = $PSCmdlet.ParameterSetName
             $p.ValueFromRemainingArguments = $true
             $p.Position = 1
             $p.Mandatory = $false
             $defaultParam.Attributes.Add($p)
             
             $dict.Add("DefaultValue", $defaultParam)
             $dict
         }
     }
     
     end {                
         $default = $pscmdlet.MyInvocation.BoundParameters["DefaultValue"]               
         
 
         $help = @{}
 
         if ($IncludeHelp) {
 
             $xmldoc = Get-XmlDocumentation -type $enumeratedType        
 
             if ($xmlDoc) {
 
                 $f = [xml][io.file]::readAllLines($xmlDoc)
 
                 $selector = "F:$($enumeratedType.fullName)"
 
                 foreach ($node in $f.doc.members.selectnodes("member[starts-with(@name,'$selector')]")) {
                     if ($node.summary) {
                         $help[$node.name] = $node.summary.trim()
                     }
                 }
             }
         }
 
         $choices = new-object collections.generic.list[System.Management.Automation.Host.ChoiceDescription]
 
         $names = [enum]::getnames($enumeratedType)
         [double[]]$values = [enum]::getvalues($enumeratedType)|%{[double]$_}
         
         $hot = new-hotkeytable $names
 
         $names | % {
             
             $i = $_.indexof([string]$hot[$_], [stringcomparison]::ordinalignorecase)
             
             if ($i -ne -1) {
             
                 $name = $_.insert($i, "&") 
             
             } else {
             
                 $name = "$_ (&$($hot[$_]))"
             }
 
             $helpKey = "F:$($enumeratedType.fullname).$_"
             
             if ($includehelp -and $help[$helpKey]) {
             
                 $choiceParams =  @($name, $help[$helpKey])
             
             } else {
             
                 $choiceParams = $name
             }
             
             $choices.add((new-object Management.Automation.Host.ChoiceDescription -Args $choiceParams))
         }
 
         $isFlags = $enumeratedType.GetCustomAttributes([FlagsAttribute], $false)
         
         $supportsMultipleChoice = $host.ui -is [Management.Automation.Host.IHostUISupportsMultipleChoiceSelection]
         
         if ($isFlags -and (-not $supportsMultipleChoice)) {
             
             throw ("{0} enum is flags decorated and this host ({1}) does not support multiple choice selections. Sorry!" -f $enumeratedType.name, $host.name)
         
         }
         
         $title = $enumeratedType.name               
 
         if ($isFlags) {
         
             if (-not $default) {
         
                 $default = [int[]]@()
         
             } else {
         
                 [int[]]$defaults = @()
                 $limit = [math]::log([enum]::GetUnderlyingType($enumeratedType)::maxvalue + 1, 2)
                 
                 for ($index = [int]$limit; $index -ge 0; $index--) {
         
         
                     if (([int]$default) -band $bit) {
                         $defaults += [array]::indexof($values, $bit)
                     }
                 }
                 $default = $defaults
             }
             
             $message = "Choose one or more values for mask:"
             $title += " (Flags)"
 
         } else {
         
             if (-not $default) {
         
                 $default = -1
         
             } else {
         
                 $default = [array]::indexof($values, [double]$default)
             }
             $message = "Choose single value:"
         }
         
         $result = @()
         
         $host.ui.promptforchoice($title, $message, $choices, $default) | % {
             $result += $names[$_]
         }
         
         $result -as $enumeratedtype
     }
 }
 
 function Get-HelpSummary
 {
         [CmdletBinding()]
         param
         (        
             [string]$file,
             [reflection.assembly]$assembly,
             [string]$selector
         )
         
         if ($helpCache.ContainsKey($assembly))
         {
             $xml = $helpCache[$assembly]            
         }
         else
         {
             Write-Progress -id 1 "Caching Help Documentation" $assembly.getname().name
 
             $xml = [xml](gc $file)
             
             $helpCache.Add($assembly, $xml)
             
             Write-Progress -id 1 "Caching Help Documentation" $assembly.getname().name -completed
         }
 
         $summary = $xml.doc.members.SelectSingleNode("member[@name='$selector' or starts-with(@name,'$selector(')]").summary
         
         $summary
 }
 
 function New-HotKeyTable {
     param([string[]]$names)
     
     $hot = @{}
 
     $names | % {
         
         $c = [char]::toupper($_[0])
         
         if (-not $hot.containsvalue($c)) {
             $hot[$_] = $c
             write-debug "1) assigned $c to $_"
         }
     }
 
     $names | ? {
 
         -not $hot.containskey($_)
 
     } | % {
         
         for ($i=1; $i -lt $_.length; $i++) {
             
             $c = $_[$i]
             
             if ([char]::IsUpper($c) -and (-not $hot.containsvalue($c))) {
                 $hot[$_] = $c
                 write-debug "2) assigned $c to $_"
                 break
             }
         }
     }
 
     $names | ? {
 
         -not $hot.containskey($_)
 
     } | % {
 
         for ($i=1; $i -lt $_.length; $i++) {
             
             $c = [char]::toupper($_[$i])
             
             if (-not $hot.containsvalue($c)) {
                 $hot[$_] = $c
                 write-debug "3) assigned $c to $_"
                 break
             }
         }
     }
 
     $names | ? {
 
         -not $hot.containskey($_)
 
     } | % { 
 
         $word = $_
 
         $s = [int]"A"[0]
         
         for ($i = $s; $i -lt ($s + 26); $i++) {
 
             $c = [char]$i
             
             if (-not $hot.containsvalue($c)) {
                 $hot[$word] = $c
                 write-debug "4) assigned $c to $word"
                 break
             }
         }
     }
     
     $hot
 }
 
 function Get-XmlDocumentation {
     [cmdletbinding()]
     param([type]$type)
     
     $docFilename = [io.path]::changeextension([io.path]::getfilename($type.assembly.location), ".xml")
     $location = [io.path]::getdirectoryname($type.assembly.location)
     $codebase = (new-object uri $type.assembly.codebase).localpath
             
     $frameworkDir = "${env:windir}\Microsoft.NET\framework\v2.0.50727"
     $lang = [system.globalization.cultureinfo]::CurrentUICulture.parent.name
 
     switch
         (
         "${frameworkdir}\${lang}\$docFilename",
         "${frameworkdir}\$docFilename",
         "$location\$docFilename",
         "$codebase\$docFilename"
         )
     {
         { test-path $_ } { $_; return; }
         
         default
         {
             continue;
         }        
     }
 }
`

