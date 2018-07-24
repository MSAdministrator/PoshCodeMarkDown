---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4765
Published Date: 2014-01-03t12
Archived Date: 2014-01-06t04
---

# expand-peinfo (fix 1) - 

## Description

first fix for expand-peinfo (http

## Comments



## Usage



## TODO



## function

`trace-binaryreader`

## Code

`#
 #
 function script:Get-Class([String]$Assembly, [String]$Class) {
   return ([AppDomain]::CurrentDomain.GetAssemblies() | ? {
     $_.Location.Split('\')[-1].Equals($Assembly)
   }).GetType($Class)
 }
 
 function script:Test-Assembly([String]$Wildcard) {
   if (-not $Wildcard.EndsWith('*')) {$Wildcard += '*'}
   
   return (([AppDomain]::CurrentDomain.GetAssemblies() | ? {
     $_.FullName -like $Wildcard
   }) -ne $null)
 }
 
 function script:Get-PEValues([String]$FileName) {
   begin {
     if (-not (Test-Assembly System.Deployment)) {
       Add-Type -Assembly System.Deployment
     }
 
     $PEStream = Get-Class System.Deployment.dll System.Deployment.Application.PEStream
     
     $IMAGE_DOS_HEADER        = [Activator]::CreateInstance(
       $PEStream.GetNestedType('IMAGE_DOS_HEADER', 'NonPublic')
     )
     $IMAGE_FILE_HEADER       = [Activator]::CreateInstance(
       $PEStream.GetNestedType('IMAGE_FILE_HEADER', 'NonPublic')
     )
     $IMAGE_OPTIONAL_HEADER32 = [Activator]::CreateInstance(
       $PEStream.GetNestedType('IMAGE_OPTIONAL_HEADER32', 'NonPublic')
     )
     
     function Trace-BinaryReader([IO.BinaryReader]$reader, [Object]$data) {
       $bytes = $br.ReadBytes(
         [Runtime.InteropServices.Marshal]::SizeOf($data)
       )
       $hndl = [Runtime.InteropServices.GCHandle]::Alloc(
         $bytes, [Runtime.InteropServices.GCHandleType]::Pinned
       )
       $struct = [Runtime.InteropServices.Marshal]::PtrToStructure(
         $hndl.AddrOfPinnedObject(), $data.GetType()
       )
       $hndl.Free()
       
       return $struct
     }
     
     function Trace-TimeStamp([Object]$stamp) {
       return [TimeZone]::CurrentTimeZone.ToLocalTime(
         ([DateTime]'1.1.1970').AddSeconds($stamp)
       )
     }
     
     function Format-Output([Object]$type) {
       $type.PSObject.Properties | % {'{0, 16} {1}' -f ('{0:X}' -f $_.Value), $_.Name}
     }
   }
   process {
     try {
       $fs = New-Object IO.FileStream($FileName, [IO.FileMode]::Open, [IO.FileAccess]::Read)
       $br = New-Object IO.BinaryReader($fs)
       
       $dos = Trace-BinaryReader $br $IMAGE_DOS_HEADER
       [void]$fs.Seek($dos.e_lfanew, [IO.SeekOrigin]::Begin)
       $sig = $br.ReadUInt32()
       
       $fh = Trace-BinaryReader $br $IMAGE_FILE_HEADER
       if ($fh.Machine -eq 0x14c) {
         $pe32 = Trace-BinaryReader $br $IMAGE_OPTIONAL_HEADER32
       }
       
     }
     finally {
       if ($br -ne $null) {$br.Close()}
       if ($fs -ne $null) {$fs.Close()}
     }
   }
   end {
     "FILE HEADER VALUES`n$('-' * 35)"
     Format-Output $fh
     "$('-' * 35)`nTime date stamp $(Trace-TimeStamp $fh.TimeDateStamp)"
     "Machine type is $(switch($fh.Machine){0x14c{'32-bit'}0x200{'Itanium'}0x8664{'64-bit'}})"
     "`n`nOPTIONAL HEADER VALUES`n$('-' * 35)"
     Format-Output $pe32
   }
 }
`

