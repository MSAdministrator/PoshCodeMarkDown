---
Author: jason ferris
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2288
Published Date: 2011-10-07t07
Archived Date: 2017-05-16t03
---

# set-encoding - 

## Description

provides an easy way to change the text encoding on a lot of scripts at once.

## Comments



## Usage



## TODO



## function

`set-encoding`

## Code

`#
 #
 function Set-Encoding{
 <#
 .Synopsis
    Takes a Script file or any other text file into memory and Re-Encodes it in the format specified.
 .Parameter Source
    The path to the file to be re-encoded.
 .Parameter Destination
    The path to write the corrected file to
 .Parameter Encoding 
    The encoding to convert to
    Note:
    Default  Uses the encoding of the system's current ANSI code page.
    OEM      Uses the current original equipment manufacturer code page identifier for the operating system.
 .Example
   ls *.ps1 | Set-Encoding ASCII
 .Example
   ls *.ps1 | Set-Encoding UTF8 -Destination { [System.IO.Path]::ChangeExtension($_.FullName, ".utf8.ps1") }
 
 .Description
    Provides an easy way to change the text encoding on a lot of scripts at once.
 .Notes
    Don't use on non-text files!
    The script just calls Out-File with the -Encoding parameter and the -Force switch, but it's very useful for making sure all your script files are correctly UTF8 or ASCII encoded so they can be signed.
 #>
 
 [CmdletBinding()]
 
 PARAM(
    [Parameter(Position=0,Mandatory=$true,ValueFromPipeline=$false)]
    [ValidateSet("Unicode","UTF7","UTF8","UTF32","ASCII", "BigEndianUnicode","Default", "OEM")]
    [string]$Encoding
 ,
    [Parameter(Position=1, Mandatory=$true, ValueFromPipelineByPropertyName=$true)]
    [ValidateScript({ 
       if((resolve-path $_).Provider.Name -ne "FileSystem") {
          throw "Specified Path is not in the FileSystem: '$_'" 
       }
       return $true
    })]
    [Alias("Fullname","Path","File")]
    [string]$Source
 ,
    [Parameter(Position=2, Mandatory=$false, ValueFromPipelineByPropertyName=$true)]
    [string]$Destination = $Source
 ,
    [Switch]$Passthru
 )
 PROCESS{
    Write-Verbose "Encoding content from '$Source' into '$Destination' with $Encoding encoding."
    (Get-Content $Source) | Out-File -FilePath $Destination -Encoding $Encoding -Force
    if($Passthru){ Get-Item $Destination }
 }
 }
`

