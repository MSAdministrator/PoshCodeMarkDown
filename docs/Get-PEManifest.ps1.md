---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 5316
Published Date: 2014-07-20t14
Archived Date: 2014-07-25t21
---

# get-pemanifest - 

## Description

script is commented for explanation of each step

## Comments



## Usage



## TODO



## function

`get-pemanifest`

## Code

`#
 #
 function Get-PEManifest {
   <#
     .SYNOPSIS
         Extract PE file manifest if it's possible.
     .DESCRIPTION
         This script is alternative for next way to get dump of PE manifest:
         Add-Type -AssemblyName System.Deployment
         
         try {
           [Text.Encoding]::Default.GetString(
             ($SystemUtils = [Activator]::CreateInstance(
               ([AppDomain]::CurrentDomain.GetAssemblies() | ? {
                  $_.FullName.Contains(($asm = 'System.Deployment'))
               }).GetType(
                  $asm + '.Application.Win32InterOp.SystemUtils'
               )
             )).GetType().InvokeMember(
               'GetManifestFromPEResources',
               [Reflection.BindingFlags]280,
               $null,
               $SystemUtils,
               @($PE)
             )
           )
         }
         catch [Management.Automation.MethodInvocationException] {
           $_.Exception.InnerException
         }
     .EXAMPLE
         PS E:\> Get-PEManifest "E:\Program Files(x86)\Sysinternals\accesschk.exe"
         <?xml version="1.0" encoding="UTF-8" standalone="yes"?>
         <assembly xmlns="urn:schemas-microsoft-com:asm.v1" manifestVersion="1.0">
         <assemblyIdentity
             name="AccessChk"
             processorArchitecture="x86"
             version="1.0.0.0"
             type="win32"/>
         <description>File System Monitor</description>
         <dependency>
             <dependentAssembly>
                 <assemblyIdentity
                     type="win32"
                     name="Microsoft.Windows.Common-Controls"
                     version="6.0.0.0"
                     processorArchitecture="x86"
                     publicKeyToken="6595b64144ccf1df"
                     language="*"
                 />
             </dependentAssembly>
         </dependency>
           <trustInfo xmlns="urn:schemas-microsoft-com:asm.v3">
             <security>
               <requestedPrivileges>
                 <requestedExecutionLevel level='asInvoker' uiAccess='false' />
               </requestedPrivileges>
             </security>
           </trustInfo>
         </assembly>
     .LINK
         http://download.microsoft.com/download/9/c/5/9c5b2167-8017-4bae-9fde-d599bac8184a/pecoff_v83.docx
     .NOTES
         Author: greg zakharov
   #>
   param(
     [Parameter(Mandatory=$true, ValueFromPipeline=$true)]
     [ValidateScript({Test-Path $_})]
     [String]$FileName
   )
   
   try {
     $fs = [IO.File]::OpenRead($FileName)
     $br = New-Object IO.BinaryReader($fs)
     
     $buf = New-Object "Byte[]" 2
     [void]$br.Read($buf, 0, $buf.Length)
     $e_magic = [Text.Encoding]::Default.GetString($buf)
     $fs.Position = 0x3C
     $e_lfanew = $br.ReadUInt16()
     [void]$fs.Seek($e_lfanew, [IO.SeekOrigin]::Begin)
     $buf = New-Object "Byte[]" 4
     [void]$br.Read($buf, 0, $buf.Length)
     $pe_sign = [Text.Encoding]::Default.GetString($buf)
     
     if ($e_magic -ne 'MZ' -and $pe_sign -ne "PE`0`0") {
       throw (New-Object ComponentModel.Win32Exception('Invalid file format.'))
     }
     
     $fs.Position += 0x02
     $pe_sec = $br.ReadUInt16()
     $fs.Position += 0x0C
     $pe_ioh = $br.ReadUInt16()
     
     $fs.Position += $pe_ioh + 0x02
     0..($pe_sec - 1) | % {
       [void]$br.Read($buf, 0, $buf.Length)
       if ([Text.Encoding]::Default.GetString($buf) -ne '.rsrc') {
       }
         $fs.Position += 0x04
         $vrt_addr = $br.ReadUInt32()
         $fs.Position += 0x04
         $ptr_data = $br.ReadUInt32()
       }
     
     if ($vrt_addr -eq $null -or $ptr_data -eq $null) {
       throw (New-Object ComponentModel.Win32Exception('File does not contain resources or packed.'))
     }
     
     $fs.Position  = $ptr_data + 0x0C
     0..($br.ReadUInt16() + $br.ReadUInt16() - 1) | % {
       $Name = $br.ReadUInt32()
       $OffsetToData = $br.ReadUInt32() -band 0xfffffff
         $raw_addr = $OffsetToData
       }
     $fs.Position = $ptr_data + 0x10 + $raw_addr
     $fs.Position += 0x04
     $raw_addr = $br.ReadUInt32() -band 0xfffffff
     $fs.Position = $ptr_data + 0x10 + $raw_addr
     $fs.Position += 0x04
     $raw_addr = $br.ReadUInt32()
     $fs.Position = $ptr_data + 0x10 + $raw_addr
     $fs.Position = $ptr_data + ($ofs_data - $vrt_addr)
     $buf = New-Object "Byte[]" $res_size
     [void]$br.Read($buf, 0, $buf.Length)
     [Text.Encoding]::Default.GetString($buf)
   }
   catch {
     if (($exp = $_.Exception.InnerException) -eq $null) {
       Write-Host $_.Exception.Message`n
     }
     else { Write-Host $exp.Message`n }
   }
   finally {
     if ($br -ne $null) { $br.Close() }
     if ($fs -ne $null) { $fs.Close() }
   }
 }
`

