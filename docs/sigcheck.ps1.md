---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 1.11
Encoding: ascii
License: cc0
PoshCode ID: 4806
Published Date: 2015-01-17t10
Archived Date: 2015-07-20t00
---

# sigcheck - 

## Description

something like sigcheck

## Comments



## Usage



## TODO



## function

`get-filesignature`

## Code

`#
 #
 Set-Alias sigcheck Get-FileSignature
 
 function Get-FileSignature {
   <#
     .SYNOPSIS
         File version and signature viewer.
     .EXAMPLE
         PS C:\>sigcheck E:\bin\whois.exe -h -m
         
 
         Verified           : Valid
         MachineType        : I386
         Owner              : GZ\Guest
         Hashes             : {MD5: 6709cd2e10b658170309a7a171c9f678, SHA1: 11d8c03dcbe4f4579d23673ae43b2957e4296799, SHA256: 0e
                              725efd84c66a246c011129ac19070da1625421ea2f893de0d3a544adaaca8b}
         Manifest           : <assembly xmlns="urn:schemas-microsoft-com:asm.v1" manifestVersion="1.0">
                                <trustInfo xmlns="urn:schemas-microsoft-com:asm.v3">
                                  <security>
                                    <requestedPrivileges>
                                      <requestedExecutionLevel level="asInvoker" uiAccess="false"></requestedExecutionLevel>
                                    </requestedPrivileges>
                                  </security>
                                </trustInfo>
                              </assembly>
         Comments           :
         CompanyName        : Sysinternals - www.sysinternals.com
         FileBuildPart      : 0
         FileDescription    : Whois - domain information lookup
         FileMajorPart      : 1
         FileMinorPart      : 11
         FileName           : E:\bin\whois.exe
         FilePrivatePart    : 0
         FileVersion        : 1.11
         InternalName       : whois
         IsDebug            : False
         IsPatched          : False
         IsPrivateBuild     : False
         IsPreRelease       : False
         IsSpecialBuild     : False
         Language           : English (USA)
         LegalCopyright     : Copyright c 2005-2012 Mark Russinovich
         LegalTrademarks    :
         OriginalFilename   : whois.exe
         PrivateBuild       :
         ProductBuildPart   : 0
         ProductMajorPart   : 1
         ProductMinorPart   : 11
         ProductName        : Sysinternals Whois
         ProductPrivatePart : 0
         ProductVersion     : 1.11
         SpecialBuild       :
         
         
         PS C:\>
   #>
   [CmdletBinding()]
   param(
     [Parameter(Mandatory=$true, Position=0, ValueFromPipeline=$true)]
     [ValidateScript({Test-Path $_})]
     [String]$FileName,
     
     [Alias('h')]
     [Switch]$HashesShow,
     
     [Alias('m')]
     [Switch]$ManifestDump
   )
   
   begin {
     $FileName = cvpa $FileName
     $asm = 'System.Deployment'
     Add-Type -Assembly $asm
     
     function Get-MachineType([String]$FileName) {
       $bytes = New-Object "Byte[]" 1024
       
       try {
         $fs = New-Object IO.FileStream($FileName, [IO.FileMode]::Open, [IO.FileAccess]::Read)
         [void]$fs.Read($bytes, 0, 1024)
         [Int32]$res = [BitConverter]::ToUInt16($bytes, ([BitConverter]::ToInt32($bytes, 0x3c) + 0x4))
       }
       catch [Management.Automation.RuntimeException] {
         $exp = [Boolean]$_.Exception
       }
       finally {
         if ($fs -ne $null) {$fs.Close()}
       }
       
       if (!$exp) {[Reflection.ImageFileMachine]$res}
     }
     
     function Get-Hashes([String]$HashKind, [String]$FileName) {
       if (([IO.FileInfo]$FileName).Length -ne 0) {
         try {
           $s = [IO.File]::OpenRead($FileName)
           [Security.Cryptography.HashAlgorithm]::Create($HashKind).ComputeHash($s) | % {
             $res = ''}{$res += $_.ToString('x2')}{'{0}: {1}' -f $HashKind, $res
           }
         }
         finally {
           if ($s -ne $null) {$s.Close()}
         }
       }
     
     function Get-PEManifest([String]$FileName) {
       begin {
         $su = ([AppDomain]::CurrentDomain.GetAssemblies() | ? {
           $_.FullName.Split(',')[0] -eq $asm
         }).GetType(($asm + '.Application.Win32InterOp.SystemUtils'))
         $a = [Activator]::CreateInstance($su)
       }
       process {
         try {
           -join [Char[]]$a.GetType().InvokeMember('GetManifestFromPEResources',
                 [Reflection.BindingFlags]280, $null, $su, @($FileName)
           )
         }
         catch [Management.Automation.RuntimeException] {}
       }
   }
   process {
     $inf = [Diagnostics.FileVersionInfo]::GetVersionInfo($FileName)
     $inf = Add-Member -mem ScriptProperty -nam Verified -inp $inf -val {
       (Get-AuthenticodeSignature $this.FileName).Status
     } -pas
     $inf = Add-Member -mem ScriptProperty -nam MachineType -inp $inf -val {
       Get-MachineType $this.FileName
     } -pas
     $inf = Add-Member -mem ScriptProperty -nam Owner -inp $inf -val {
       ([IO.FileInfo]$this.FileName).GetAccessControl().Owner
     } -pas
     
     if ($HashesShow) {
       $inf = Add-Member -mem ScriptProperty -nam Hashes -inp $inf -val {
         'MD5', 'SHA1', 'SHA256' | % {Get-Hashes $_ $this.FileName}
       } -pas
     }
     
     if ($ManifestDump) {
       $inf = Add-Member -mem ScriptProperty -nam Manifest -inp $inf -val {
         Get-PEManifest $this.FileName
       } -pas
     }
   }
   end {$inf | fl *}
 }
`

