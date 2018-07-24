---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4764
Published Date: 2014-01-03t09
Archived Date: 2014-01-04t21
---

# expand-peinfo - 

## Description

this is just a preliminary version

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
     
     $IMAGE_DOS_HEADER     = [Activator]::CreateInstance(
       $PEStream.GetNestedType('IMAGE_DOS_HEADER', 'NonPublic')
     )
     $IMAGE_FILE_HEADER    = [Activator]::CreateInstance(
       $PEStream.GetNestedType('IMAGE_FILE_HEADER', 'NonPublic')
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
   }
   process {
     try {
       $fs = New-Object IO.FileStream($FileName, [IO.FileMode]::Open, [IO.FileAccess]::Read)
       $br = New-Object IO.BinaryReader($fs)
       
       $dos = Trace-BinaryReader $br $IMAGE_DOS_HEADER
       [void]$fs.Seek($dos.e_lfanew, [IO.SeekOrigin]::Begin)
       $sig = $br.ReadUInt32()
       
       $fh = Trace-BinaryReader $br $IMAGE_FILE_HEADER
     }
     finally {
       if ($br -ne $null) {$br.Close()}
       if ($fs -ne $null) {$fs.Close()}
     }
   }
   end {
     "FILE HEADER VALUES`n$('-' * 35)"
     $fh.PSObject.Properties | % {'{0, 16} {1}' -f ('{0:X}' -f $_.Value), $_.Name}
     "$('-' * 35)`nTime date stamp $(Trace-TimeStamp $fh.TimeDateStamp)"
     "Machine type is $(switch($fh.Machine){0x14c{'32-bit'}0x200{'Itanium'}0x8664{'64-bit'}})"
     "`n"
   }
 }
 
 function script:Get-PEManifest([String]$FileName) {
   begin {
     if (-not (Test-Assembly System.Deployment)) {
       Add-Type -Assembly System.Deployment
     }
     
     $SystemUtils = Get-Class System.Deployment.dll System.Deployment.Application.Win32InterOp.SystemUtils
     $SystemUtilsClass = [Activator]::CreateInstance($SystemUtils)
   }
   process {
     try {
       $res = -join ($SystemUtilsClass.GetType().InvokeMember('GetManifestFromPEResources',
                            [Reflection.BindingFlags]'Public, Static, InvokeMethod', $null,
                                            $SystemUtilsType, @($FileName)) | % {[Char]$_})
     }
     catch {}
   }
   end {
     "FILE MANIFEST DUMP`n$('-' * 35;"`n";if ($res -ne '') {$res} else {'n/a'})"
     "`n"
   }
 }
 
 function Expand-PEInfo {
   [CmdletBinding()]
   param(
     [Parameter(Mandatory=$true, Position=0, ValueFromPipeline=$true)]
     [ValidateScript({Test-Path $_})]
     [String]$FileName,
     
     [Switch]$Values,
     
     [Switch]$Manifest,
     
     [Switch]$Attributes
   )
   
   begin {
     $FileName = cvpa $Filename
     $inf = gci $FileName
   }
   process {
     "FILE VERSION INFO`n$('-' * 35)"
     $inf.VersionInfo | % {'{0}' -f $_}
     ''
     
     if ($Values) {Get-PEValues $FileName}
     if ($Manifest) {Get-PEManifest $FileName}
     if ($Attributes) {
       "FILE ATTRIBUTES`n$('-' * 35)"
       [Enum]::GetValues([IO.FileAttributes]) | % {
         '{0, 19} {1}' -f $_, $([Boolean]([IO.FileAttributes]::$_ -band $inf.Attributes))
       }
       "`n"
     }
   }
   end {
   }
 }
`

