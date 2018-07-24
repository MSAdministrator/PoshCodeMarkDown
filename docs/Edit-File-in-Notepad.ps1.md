---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 808
Published Date: 2009-01-15t12
Archived Date: 2016-03-22t13
---

# edit-file in notepad++ - 

## Description

a script to open files for editing in notepad++ — you may need to edit the $npp variable in the begin block to put the full path in, if you didn’t run the notepad++ installer.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 .Synopsis 
    Open a file for editing in notepad++
 .Description
    Opens one or more files in Notepad++, passing all the switches each time.  Accepts filenames on the pipeline
 .Notes 
    I took the "no" off the parameters, because in PowerShell you only need to enter just enough of the parameter name to differentiate ... this way each one is differentiatable by the first letter.
 .Link
    http://notepad-plus.sourceforge.net/uk/cmdLine-HOWTO.php
 .Link
    http://notepad-plus.sourceforge.net/
 .Link
    http://poshcode.org/notepad++lexer/
 .Example
    Edit-File $Profile -MultiInstance -SessionOff -TabBarOff -PluginsOff
    
    Open your Profile in a new Notepad++ window in a "notepad" like mode: multi-session, no tab bars, no previous files, no plugins.
    
 .Example
    ls *.ps1 | Edit-File -SessionOff
    
    Open all the ps1 scripts in Notepad++ without restoring previous files.
    
 .Example
    Edit-File -language xml ~\Projects\Project1\Project1.csproj
 
    Open the file Project1.csproj as a xml file, even though its extension is not recognized as xml...
    
 .Parameter File
    The absolute or relative path of file(s) you want to open, accepts wildcards
 .Parameter MultiInstance
    Notepad++ is unique instance by default. This option allows you to launch several instances of Notepad++.
 .Parameter PluginsOff
    Use this parameter to launch Notepad++ without loading the plugins. This parameter is useful when you want to determinate where the bad behaviour or crash come from (from the plugins or from Notepad++ itself).
 .Parameter Language
    This option applies the specified Language to the filesToOpen to open, allowing you to specify XML for csproj files, for example.
    Valid Languages are : c, cpp, java, cs, objc, rc, html, javascript, php, vb, sql, xml, asp, perl, pascal, python, css, lua, batch, ini, nfo, tex, fortran, bash, actionscript and nsis.
 .Parameter Number
    Cause Notepad++ to go to the LineNumber you want after opening the file
 .Parameter SessionOff
    Launch Notepad++ without loading the previous session (the files opened in Notepad++ last time).
 .Parameter ReadOnly
    Open files in read only mode. (Alias: RO)
 .Parameter TabBarOff
    Turn off the tab interface. Useful if you want a minimalist session ...
 #>
 [CmdletBinding()]
 Param(
    [Parameter(ValueFromPipelineByPropertyName=$true,Position=1)]
    [Alias("FullName","LiteralPath","Path")]
    [string[]]
    $File
 ,
    [Parameter()]
    [switch]$MultiInstance
 ,
    [Parameter()]
    [switch]$PluginsOff
 ,
    [Parameter()]
    [string]$Language
 ,
    [Parameter(Position=2)]
    [int]$Number 
 ,
    [Parameter()]
    [switch]$SessionOff
 ,
    [Parameter()]
    [Alias("RO")]
    [Switch]$ReadOnly
 ,
    [Parameter()]
    [Switch]$TabBarOff
 )
 BEGIN {
    $npp = "C:\Programs\DevTools\Notepad++\notepad++.exe"
    $param = @( 
       if($MultiInstance) { "-multiInst" }
       if($PluginsOff)    { "-noPlugin" }
       if($Language)      { "-l$Language" }
       if($Number)        { "-n$Number" }
       if($SessionOff)    { "-nosession" }
       if($ReadOnly)      { "-ro" }
       if($TabBarOff)     { "-notabbar" }
       " "
    ) -join " "
 }
 PROCESS {    
    foreach($path in $File) { 
       foreach($f in Convert-Path (Resolve-Path $path)) {
          $parameters = $param + """" +  + """"
          write-verbose "$npp $parameters"
          [Diagnostics.Process]::Start($npp,$parameters).WaitForInputIdle(500)
       } 
    }
 }
`

