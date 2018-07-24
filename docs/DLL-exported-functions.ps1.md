---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5028
Published Date: 2014-03-29t12
Archived Date: 2017-04-18t05
---

# dll exported functions - 

## Description

this example demonstrates how you can get the list of exported functions from the dll. be warned! this test file for x86 only. for x64 offsets will be different.

## Comments



## Usage



## TODO



## function

`sync-bytes`

## Code

`#
 #
 try {
   $fs  = [IO.File]::OpenRead((gcm -c Application ntdll.dll).Definition)
   $buf = New-Object "Byte[]" $fs.Length
   [void]$fs.Read($buf, 0, $buf.Length)
 }
 catch { $e = [Boolean]$_ }
 finally {
   if ($fs -ne $null) { $fs.Close() }
   if ($e) {
     Write-Warning 'Access denied.'
     break
   }
 }
 
 function Sync-Bytes([Int32]$offset) {
   return [BitConverter]::ToUInt32($buf, ((256 * $buf[0x3D] + $buf[0x3C]) + $offset))
 }
 
 Add-Type -TypeDefinition @'
 using System;
 using System.Reflection;
 using System.Runtime.InteropServices;
 
 [assembly: AssemblyVersion("2.0.0.0")]
 [assembly: CLSCompliant(true)]
 [assembly: ComVisible(false)]
 
 namespace PEStream {
   public sealed class Exports {
     private Exports() {}
     
     [StructLayout(LayoutKind.Sequential, Pack = 1)]
     internal struct IMAGE_EXPORT_DIRECTORY {
       public UInt32 Characteristics;
       public UInt32 TimeDateStamp;
       public UInt16 MajorVersion;
       public UInt16 MinorVersion;
       public UInt32 Name;
       public UInt32 Base;
       public UInt32 NumberOfFunctions;
       public UInt32 NumberOfNames;
       public UInt32 AddressOfFunctions;
       public UInt32 AddressOfNames;
       public UInt32 AddressOfOrdinals;
     }
   }
 }
 '@ -WarningAction 0
 
 $IMAGE_EXPORT_DIRECTORY = [Activator]::CreateInstance(
   [PEStream.Exports].GetNestedType('IMAGE_EXPORT_DIRECTORY', [Reflection.BindingFlags]32)
 )
 $exp = [Runtime.InteropServices.Marshal]::PtrToStructure(
   [IntPtr](($ImageBase = Sync-Bytes 0x34) + (Sync-Bytes 0x78)), $IMAGE_EXPORT_DIRECTORY.GetType()
 )
 0..($exp.NumberOfNames - 1) | % {
   [Runtime.InteropServices.Marshal]::PtrToStringAnsi(
     [IntPtr]($ImageBase + [Runtime.InteropServices.Marshal]::ReadInt32(
       ($ImageBase + $exp.AddressOfNames) + $_ * 4
     ))
   )
 }
`

