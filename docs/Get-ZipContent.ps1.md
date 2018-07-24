---
Author: dana james
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6136
Published Date: 2016-12-13t16
Archived Date: 2016-03-19t08
---

# get-zipcontent - 

## Description

found in my personal scripts archive (author

## Comments



## Usage



## TODO



## function

`get-zipcontent`

## Code

`#
 #
 function Get-ZipContent {
   <#
     .SYNOPSIS
         Shows contents of the specified ZIP archive.
     .DESCRIPTION
         The function does not use any third party libraries that's why you
         shouldn't worry about dependencies.
     .PARAMETER Path
         The path to the ZIP archive.
     .EXAMPLE
         PS C:\> Get-ZipContent E:\*\SysinternalsSuite.zip
     .NOTES
         Notes: greg zakharov
   #>
   [CmdletBinding(DefaultParameterSetName='Path')]
   param(
     [Parameter(Mandatory=$true, ParameterSetName='Path', Position=0,
             ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
     [ValidateScript({Test-Path $_})]
     [String]$Path,
     
     [Parameter(Mandatory=$true, ParameterSetName='LiteralPath', Position=0,
                                      ValueFromPipelineByPropertyName=$true)]
     [ValidateScript({Test-Path $_})]
     [String]$LiteralPath
   )
   
   begin {
     Set-StrictMode -Version 2.0
     
     function private:Read-Item([String]$Zip) {
       $Methods = @{Store = 0;Deflate = 8;BZIP2 = 12;LZMA = 14}
       
       function private:ConvertTo-DateTime([UInt16[]]$Stamp) {
         New-Object DateTime(
           ([Math]::Floor($Stamp[0] / [Math]::Pow(2, 9)) + 1980),
           ([Math]::Floor($Stamp[0] / [Math]::Pow(2, 5)) -band 0xF),
           ($Stamp[0] -band 0x1F),
           ([Math]::Floor($Stamp[1] / [Math]::Pow(2, 11))),
           ([Math]::Floor($Stamp[1] / [Math]::Pow(2, 5)) -band 0x3F),
           (($Stamp[1] -band 0x1F) * 2)
         )
       
       try {
         $fs = [IO.File]::OpenRead($Zip)
         $br = New-Object IO.BinaryReader($fs)
         $fs.Position = $fs.Length - 22
         $fs.Position += 16
         $fs.Position = $br.ReadUInt32()
         while ($true) {
           if ($br.ReadUInt32() -ne 0x02014b50) {break}
           $a, $b, $c = $br.ReadUInt16(), $br.ReadUInt16(), $br.ReadUInt16()
           $d, $e, $f = $br.ReadUInt32(), $br.ReadUInt32(), $br.ReadUInt32()
           $fs.Position += 4
           $CentralDirectoryFileHeader = New-Object PSObject -Property @{
             Path = -join $br.ReadChars($g)
             Method = $Methods.Keys | Where-Object {$Methods.Item($_) -eq $a}
             Attributes = [IO.FileAttributes]$j
             Crc32 = "0x$($d.ToString('X8'))"
             Modified = ConvertTo-DateTime(@($c, $b))
             Packed = $e
             Size = $f
           }
           $CentralDirectoryFileHeader.PSObject.TypeNames.Insert(0, 'CentralDirectoryFileHeader')
           $CentralDirectoryFileHeader |
           Select-Object Path, Method, Attributes, Crc32, Modified, Packed, Size
           $fs.Position += $h + $i
         }
       }
       catch { Write-Debug $_.Exception }
       finally {
         if ($br -ne $null) { $br.Close() }
         if ($fs -ne $null) { $fs.Close() }
       }
   }
   process {
     Read-Item $(switch ($PSCmdlet.ParameterSetName) {
       'Path'        { Resolve-Path $Path }
       'LiteralPath' { $LiteralPath }
     })
   }
   end {}
 }
`

