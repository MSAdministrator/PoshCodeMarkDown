---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5262
Published Date: 2014-06-25t17
Archived Date: 2014-07-04t02
---

# read-dllexportfunctions - 

## Description

prints dll exported functions list.

## Comments



## Usage



## TODO



## function

`read-dllexportfunctions`

## Code

`#
 #
 if (!(Test-Path alias:dllexp)) { Set-Alias dllexp Read-DllExportFunctions }
 
 function Read-DllExportFunctions {
   <#
     .NOTES
         Author: greg zakharov
   #>
   param(
     [Parameter(Mandatory=$true, ValueFromPipeline=$true)]
     [ValidateScript({Test-Path $_})]
     [String]$FileName
   )
   
   begin {
     $cd = [AppDomain]::CurrentDomain
     $attr = 'AnsiClass, Class, Public, SequentialLayout, Sealed, BeforeFieldInit'
     
     if (!($cd.GetAssemblies() | ? {
       $_.FullName.Contains('ExpBrowse')
     })) {
       $type = (($cd.DefineDynamicAssembly(
         (New-Object Reflection.AssemblyName('ExpBrowse')), [Reflection.Emit.AssemblyBuilderAccess]::Run
       )).DefineDynamicModule('ExpBrowse', $false)).DefineType('IMAGE_EXPORT_DIRECTORY', $attr)
       [void]$type.DefineField('Characteristics',    [UInt32], 'Public')
       [void]$type.DefineField('TimeDateStamp',      [UInt32], 'Public')
       [void]$type.DefineField('MajorVersion',       [UInt16], 'Public')
       [void]$type.DefineField('MinorVersion',       [UInt16], 'Public')
       [void]$type.DefineField('Name',               [UInt32], 'Public')
       [void]$type.DefineField('Base',               [UInt32], 'Public')
       [void]$type.DefineField('NumberOfFunctions',  [UInt32], 'Public')
       [void]$type.DefineField('NumberOfNames',      [UInt32], 'Public')
       [void]$type.DefineField('AddressOfFunctions', [UInt32], 'Public')
       [void]$type.DefineField('AddressOfNames',     [UInt32], 'Public')
       [void]$type.DefineField('AddressOfOrdinals',  [UInt32], 'Public')
       $global:IMAGE_EXPORT_DIRECTORY = $type.CreateType()
     }
   }
   process {
     try {
       $fs = [IO.File]::OpenRead($FileName)
       $buf = New-Object "Byte[]" $fs.Length
       [void]$fs.Read($buf, 0, $buf.Length)
       $e_magic = -join [Char[]]$buf[0..1]
       $e_lfanew = 256 * $buf[0x3D] + $buf[0x3C]
       $pe_sign = -join [Char[]]$buf[$e_lfanew..($e_lfanew + 3)]
       
       if ($e_magic -ne 'MZ' -or $pe_sign -ne "PE`0`0") {
         throw (New-Object FormatException('Invalid file format.'))
       }
       function Sync-Bytes([Int32]$offset) {
         return [BitConverter]::ToInt32($buf, ($e_lfanew + $offset))
       }
       
       $is32bit = ((Sync-Bytes 0x16) -band 0x0100) -eq 0x0100
       
       if (((Sync-Bytes 0x16) -band 0x2000) -eq 0x2000) {
         $ImageBase = if ($is32bit) {
           Sync-Bytes 0x34
         } else {
           [BitConverter]::ToInt64($buf, ($e_lfanew + 0x30))
         }
       }
       $ied = [Activator]::CreateInstance($IMAGE_EXPORT_DIRECTORY)
       [Runtime.InteropServices.Marshal]::PtrToStructure(
         [IntPtr]($ImageBase + (Sync-Bytes 0x78)), $ied
       )
       0..($ied.NumberOfNames - 1) | % {
         [Runtime.InteropServices.Marshal]::PtrToStringAnsi(
           [IntPtr]($ImageBase + [Runtime.InteropServices.Marshal]::ReadInt32(
             ($ImageBase + $ied.AddressOfNames) + $_ * 4
           ))
         )
       }
     }
     catch { $e = [Boolean]$_ }
     finally {
       if ($fs -ne $null) { $fs.Close() }
       if ($e) { return }
     }
   }
   end {''}
 }
`

