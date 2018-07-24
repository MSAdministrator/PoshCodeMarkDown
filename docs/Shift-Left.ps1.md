---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 1729
Published Date: 
Archived Date: 2016-03-10t10
---

#  - 

## Description

my old bytes module with some extra functions from sans all wrapped up into a single module.

## Comments



## Usage



## TODO



## class

`shift-left`

## Code

`#
 #
 Add-Type @"
 public class Shift {
    public static int   Right(int x,   int count) { return x >> count; }
    public static uint  Right(uint x,  int count) { return x >> count; }
    public static long  Right(long x,  int count) { return x >> count; }
    public static ulong Right(ulong x, int count) { return x >> count; }
    public static int    Left(int x,   int count) { return x << count; }
    public static uint   Left(uint x,  int count) { return x << count; }
    public static long   Left(long x,  int count) { return x << count; }
    public static ulong  Left(ulong x, int count) { return x << count; }
 }                    
 "@
 
 
 
 #.Example 
 #.Example 
 function Shift-Left {
 PARAM( $x=1, $y )
 BEGIN {
    if($y) {
       [Shift]::Left( $x, $y )
    }
 }
 PROCESS {
    if($_){
       [Shift]::Left($_, $x)
    }
 }
 }
 
 
 #.Example 
 #.Example 
 function Shift-Right {
 PARAM( $x=1, $y )
 BEGIN {
    if($y) {
       [Shift]::Right( $x, $y )
    }
 }
 PROCESS {
    if($_){
       [Shift]::Right($_, $x)
    }
 }
 }function Write-FileByte {
 ################################################################
 #.Synopsis
 #.Parameter ByteArray
 #.Parameter Path
 #.Example
 #.Example
 ################################################################
 [CmdletBinding()] Param (
  [Parameter(Mandatory = $True, ValueFromPipeline = $True)] [System.Byte[]] $ByteArray,
  [Parameter(Mandatory = $True)] $Path
 )
 
 if ($Path -is [System.IO.FileInfo])
  { $Path = $Path.FullName }
  { $Path = "$pwd" + "\" + "$Path" }
  { $Path = $Path -replace "^\.",$pwd.Path }
  { $Path = $Path -replace "^\.\.",$(get-item $pwd).Parent.FullName }
 else
  { throw "Cannot resolve path!" }
 [System.IO.File]::WriteAllBytes($Path, $ByteArray)
 }
 function Convert-HexStringToByteArray {
 ################################################################
 #.Synopsis
 #.Parameter String
 ################################################################
 [CmdletBinding()]
 Param ( [Parameter(Mandatory = $True, ValueFromPipeline = $True)] [String] $String )
 
 $String = $String.ToLower() -replace '[^a-f0-9\\\,x\-\:]',''
 
 $String = $String -replace '0x|\\x|\-|,',':'
 
 $String = $String -replace '^:+|:+$|x|\\',''
 
 if ($String.Length -eq 0) { ,@() ; return } 
 
 if ($String.Length -eq 1)
 { ,@([System.Convert]::ToByte($String,16)) }
 elseif (($String.Length % 2 -eq 0) -and ($String.IndexOf(":") -eq -1))
 { ,@($String -split '([a-f0-9]{2})' | foreach-object { if ($_) {[System.Convert]::ToByte($_,16)}}) }
 elseif ($String.IndexOf(":") -ne -1)
 { ,@($String -split ':+' | foreach-object {[System.Convert]::ToByte($_,16)}) }
 else
 { ,@() }
 }
 function Convert-ByteArrayToHexString {
 ################################################################
 #.Synopsis
 #.Parameter ByteArray
 #.Parameter Width
 #.Parameter Delimiter
 #.Parameter Prepend
 #.Parameter AddQuotes
 #.Example
 #
 #.Example
 #
 ################################################################
 [CmdletBinding()] Param (
  [Parameter(Mandatory = $True, ValueFromPipeline = $True)] [System.Byte[]] $ByteArray,
  [Parameter()] [Int] $Width = 10,
  [Parameter()] [String] $Delimiter = ",0x",
  [Parameter()] [String] $Prepend = "",
  [Parameter()] [Switch] $AddQuotes
 )
 
 if ($Width -lt 1) { $Width = 1 }
 if ($ByteArray.Length -eq 0) { Return }
 $FirstDelimiter = $Delimiter -Replace "^[\,\;\:\t]",""
 $From = 0
 $To = $Width - 1
 Do
 {
  $String = [System.BitConverter]::ToString($ByteArray[$From..$To])
  $String = $FirstDelimiter + ($String -replace "\-",$Delimiter)
  if ($AddQuotes) { $String = '"' + $String + '"' }
  if ($Prepend -ne "") { $String = $Prepend + $String }
  $String
  $From += $Width
  $To += $Width
 } While ($From -lt $ByteArray.Length)
 }
 function Convert-ByteArrayToString {
 ################################################################
 #.Synopsis
 #.Parameter ByteArray
 #.Parameter Encoding
 ################################################################
 [CmdletBinding()] Param (
  [Parameter(Mandatory = $True, ValueFromPipeline = $True)] [System.Byte[]] $ByteArray,
  [Parameter()] [String] $Encoding = "ASCII"
 )
 
 switch ( $Encoding.ToUpper() )
 {
  "ASCII"Â Â  { $EncodingType = "System.Text.ASCIIEncoding" }
  "UNICODE" { $EncodingType = "System.Text.UnicodeEncoding" }
  "UTF7"Â Â Â  { $EncodingType = "System.Text.UTF7Encoding" }
  "UTF8"Â Â Â  { $EncodingType = "System.Text.UTF8Encoding" }
  "UTF32"Â Â  { $EncodingType = "System.Text.UTF32Encoding" }
  DefaultÂ Â  { $EncodingType = "System.Text.ASCIIEncoding" }
 }
 $Encode = new-object $EncodingType
 $Encode.GetString($ByteArray)
 }
 function Get-FileHex {
 ################################################################
 #.Synopsis
 #.Parameter Path
 #.Parameter Width
 #.Parameter Count
 #.Parameter NoOffset
 #.Parameter NoText
 ################################################################
 [CmdletBinding()] Param (
  [Parameter(Mandatory = $True, ValueFromPipelineByPropertyName = $True)]
  [Alias("FullName","FilePath")] $Path,
  [Parameter()] [Int] $Width = 16,
  [Parameter()] [Int] $Count = -1,
  [Parameter()] [Switch] $NoOffset,
  [Parameter()] [Switch] $NoText
 )
 
 
 get-content $path -encoding byte -readcount $width -totalcount $count |
 foreach-object {
  $paddedhex = $asciitext = $null
 
  foreach ($byte in $bytes) {
  } 
 
  if ($paddedhex.length -lt $width * 3)
  { $paddedhex = $paddedhex.PadRight($width * 3," ") }
 
  foreach ($byte in $bytes) {
  if ( [Char]::IsLetterOrDigit($byte) -or
  [Char]::IsPunctuation($byte) -or
  [Char]::IsSymbol($byte) )
  { $asciitext += [Char] $byte }
  else
  { $asciitext += $placeholder }
  }
 
 
  if (-not $NoOffset) { $paddedhex = "$offsettext $paddedhex" }
  if (-not $NoText) { $paddedhex = $paddedhex + $asciitext }
  $paddedhex
 }
 }
 function Toggle-Endian {
 ################################################################
 #.Synopsis
 #.Parameter ByteArray
 #.Parameter SubWidthInBytes
 #.Example
 #.Example
 ################################################################
 [CmdletBinding()] Param (
  [Parameter(Mandatory = $True, ValueFromPipeline = $True)] [System.Byte[]] $ByteArray,
  [Parameter()] [Int] $SubWidthInBytes = 1
 )
 
 if ($ByteArray.count -eq 1 -or $ByteArray.count -eq 0) { $ByteArray ; return } 
 
 if ($SubWidthInBytes -eq 1) { [System.Array]::Reverse($ByteArray); $ByteArray ; return } 
 
 if ($ByteArray.count % $SubWidthInBytes -ne 0)
 { throw "ByteArray size must be an even multiple of SubWidthInBytes!" ; return } Â 
 
 $newarray = new-object System.Byte[] $ByteArray.count 
 
 for ($($i = 0; $j = $newarray.count - 1) ;
  $i -lt $ByteArray.count ;
  $($i += $SubWidthInBytes; $j -= $SubWidthInBytes))
 {
  for ($k = 0 ; $k -lt $SubWidthInBytes ; $k++)
  { $newarray[$j - ($SubWidthInBytes - 1) + $k] = $ByteArray[$i + $k] }
 }
 $newarray
 }
 function PushToTcpPort
 {
  param ([Byte[]] $bytearray, [String] $ipaddress, [Int32] $port)
  $tcpclient = new-object System.Net.Sockets.TcpClient($ipaddress, $port) -ErrorAction "SilentlyContinue"
  trap { "Failed to connect to $ipaddress`:$port" ; return }
  $networkstream = $tcpclient.getstream()
  $networkstream.write($bytearray,0,$bytearray.length)
  $tcpclient.close()
 }
`

