---
Author: justin dearing
Publisher: 
Copyright: 
Email: 
Version: 0.48.2
Encoding: ascii
License: mitl
PoshCode ID: 3864
Published Date: 2013-01-05t11
Archived Date: 2013-01-09t02
---

# inkscape utilities - 

## Description

cmdlets that gets the full path to inkscape.com or inkscape.exe and the directory inkscape is installed to.

## Comments



## Usage



## TODO



## script

`get-inkscapedir`

## Code

`#
 #
 <#
 .SYNOPSIS 
 
 Inkscape utility cmdlets
 
 MIT License
 Copyright (c) 2012-2013 Justin Dearing <zippy1981@gmail.com>
 
 Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:
 
 The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.
 
 THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
 #>
 
 <#
 .SYNOPSIS 
     Gets the directory Inkscape is installed to. 
 .DESCRIPTION 
     Returns the directory  Inkscape is installed to.
 .INPUTS 
     None 
 .OUTPUTS 
     System.String
 #>
 function Get-InkscapeDir() {
     if ([IntPtr]::Size -eq 8) {
 	    $inkscapeDir = (Get-ItemProperty HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\Inkscape InstallDir).InstallDir
     } else {
 	    $inkscapeDir = (Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\Inkscape InstallDir).InstallDir
     }
 
     if ([string]::IsNullOrEmpty($inkscapeDir)) {
 	    throw (New-Object IO.FileNotFoundException "Inkscape does not appear to be installed")
     } 
     if (-not (Test-Path -PathType Container $inkscapeDir)) {
 	    throw (New-Object IO.FileNotFoundException "InkScape folder $($inkscapeDir) not found", $inkscapeDir)
     }
     Write-Verbose "Inkscape install path $($installDir)"
     $inkscapeDir
 }
 
 <#
 .SYNOPSIS 
     Gets the path of inkscape.com,inkscapec.exe or inkscape.exe. 
 .DESCRIPTION 
     Returns the full path of the inkscape executable. This is one of three options:
     * inkscape.com: This is a wrapper to inkscape.exe introduced in Inkscape 0.48 that writes the output of inkscape.exe to the console.
     * inkscapec.exe: This is a crude first attempt at a wrapper. Source and description is available at http://kaioa.com/node/63.
     * inkscape.exe: This is the inkscape executable compiled as a windows app. This supports command line options and non-interactive running, but does not write any output to stdout or stderr
 .INPUTS 
     None 
 .OUTPUTS 
     System.String
  #>
 function Get-InkscapeExePath() {
     param (
         [Parameter(Mandatory=$false, Position=0)][AllowEmptyString()][string] $InkscapeDir = $(
             if ([string]::IsNullOrEmpty($InkscapeDir)) { Get-InkscapeDir } else { $InkscapeDir }
         )
     )
     $inkscapeExe = Join-Path $InkscapeDir "inkscape.exe"
     if (-not (Test-Path -PathType Leaf $inkscapeExe))  {
 	    throw (New-Object IO.FileNotFoundException "InkScape.exe not found", $inkscapeExe)
     }
     if (Test-Path -PathType Leaf (Join-Path $inkscapeDir "inkscape.com")) {
         $inkscapeExe = Join-Path $InkscapeDir "inkscape.com"
     }
     else {
 	    
         if (Test-Path -PathType Leaf (Join-Path $inkscapeDir "inkscapec.exe")) {
 	        Write-Warning "inkscapec.exe found. You should probably upgrade to inkscape 0.48.2 which uses inkscape.com"
             $inkscapeExe = Join-Path $inkscapeDir "inkscapec.exe"
         }
         else {
             Write-Warning 'inkscape.com not found. Images *should* be generated, but inkscape.exe will not write anything to stdout or stderr'
         }
     }
 	    
     Write-Verbose "InkScape executablepath: $inkscapeExe"
     $inkscapeExe
 }
`

