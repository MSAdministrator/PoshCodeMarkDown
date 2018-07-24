---
Author: danward
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3252
Published Date: 2012-02-21t19
Archived Date: 2016-04-15t23
---

# file encoding no bom - 

## Description

returns the encoding type of the file; uses byte order marker (bom) if exists else analyzes files contents to determine encoding.

## Comments



## Usage



## TODO



## function

`get-dtwfileencoding`

## Code

`#
 #
 
 <#
 .SYNOPSIS
 Returns the encoding type of the file
 .DESCRIPTION
 Returns the encoding type of the file.  It first attempts to determine the 
 encoding by detecting the Byte Order Marker using Lee Holmes' algorithm
 (http://poshcode.org/2153).  However, if the file does not have a BOM
 it makes an attempt to determine the encoding by analyzing the file content
 (does it 'appear' to be UNICODE, does it have characters outside the ASCII
 range, etc.).  If it can't tell based on the content analyzed, then 
 it assumes it's ASCII. I haven't checked all editors but PowerShell ISE and 
 PowerGUI both create their default files as non-ASCII with a BOM (they use
 Unicode Big Endian and UTF-8, respectively).  If your file doesn't have a 
 BOM and 'doesn't appear to be Unicode' (based on my algorithm*) but contains 
 non-ASCII characters after index ByteCountToCheck, the file will be incorrectly
 identified as ASCII.  So put a BOM in there, would ya!
 
 For more information and sample encoding files see:
 http://danspowershellstuff.blogspot.com/2012/02/get-file-encoding-even-if-no-byte-order.html
 And please give me any tips you have about improving the detection algorithm.
 
 *For a full description of the algorithm used to analyze non-BOM files, 
 see "Determine if Unicode/UTF8 with no BOM algorithm description".
 .PARAMETER Path
 Path to file
 .PARAMETER ByteCountToCheck
 Number of bytes to check, by default check first 10000 character.
 Depending on the size of your file, this might be the entire content of your file.
 .PARAMETER PercentageMatchUnicode
 If pecentage of null 0 value characters found is greater than or equal to
 PercentageMatchUnicode then this file is identified as Unicode.  Default value .5 (50%)
 .EXAMPLE
 Get-IHIFileEncoding -Path .\SomeFile.ps1 1000
 Attempts to determine encoding using only first 1000 characters
 BodyName          : unicodeFFFE
 EncodingName      : Unicode (Big-Endian)
 HeaderName        : unicodeFFFE
 WebName           : unicodeFFFE
 WindowsCodePage   : 1200
 IsBrowserDisplay  : False
 IsBrowserSave     : False
 IsMailNewsDisplay : False
 IsMailNewsSave    : False
 IsSingleByte      : False
 EncoderFallback   : System.Text.EncoderReplacementFallback
 DecoderFallback   : System.Text.DecoderReplacementFallback
 IsReadOnly        : True
 CodePage          : 1201
 #>
 function Get-DTWFileEncoding {
   [CmdletBinding()]
   param(
     [Parameter(Mandatory = $true,ValueFromPipeline = $true,ValueFromPipelineByPropertyName = $true)]
     [ValidateNotNullOrEmpty()]
     [Alias("FullName")]
     [string]$Path,
     [Parameter(Mandatory = $false)]
     [int]$ByteCountToCheck = 10000,
     [Parameter(Mandatory = $false)]
     [decimal]$PercentageMatchUnicode = .5
   )
   process {
     [int]$MinCharactersToCheck = 400
     if ($false -eq (Test-Path -Path $Path)) {
       Write-Error -Message "$($MyInvocation.MyCommand.Name) :: Path does not exist: $Path"
       return
     }
     if ($ByteCountToCheck -lt $MinCharactersToCheck) {
       Write-Error -Message "$($MyInvocation.MyCommand.Name) :: ByteCountToCheck should be at least $MinCharactersToCheck : $ByteCountToCheck"
       return
     }
 
     $Unknown = "UNKNOWN"
     $result = $Unknown
 
     $encodings = @{}
 
     $encodingMembers = [System.Text.Encoding] | Get-Member -Static -MemberType Property
     $encodingMembers | ForEach-Object {
       $encodingBytes = [System.Text.Encoding]::($_.Name).GetPreamble() -join '-'
       $encodings[$encodingBytes] = $_.Name
     }
 
     $encodingLengths = $encodings.Keys | Where-Object { $_ } | ForEach-Object { ($_ -split "-").Count }
 
     foreach ($encodingLength in $encodingLengths | Sort-Object -Descending) {
       $bytes = (Get-Content -Path $Path -Encoding byte -ReadCount $encodingLength)[0]
       $encoding = $encodings[$bytes -join '-']
 
       if ($encoding) {
         $result = $encoding
         break
       }
     }
     if ($result -ne $Unknown) {
       [System.Text.Encoding]::$result
       return
     }
 
     <#
        Looking at the content of many code files, most of it is code or
        spaces.  Sure, there are comments/descriptions and there are variable
        names (which could be double-byte characters) or strings but most of
        the content is code - represented as single-byte characters.  If the
        file is Unicode but the content is mostly code, the single byte
        characters will have a null/value 0 byte as either as the first or
        second byte in each group, depending on Endian type.
        My algorithm uses the existence of these 0s:
         - look at the first ByteCountToCheck bytes of the file
         - if any character is greater than 127, note it (if any are found, the 
           file is at least UTF8)
         - count the number of 0s found (in every other character)
             null/value 0, then assume it is Unicode
           - if the percentage of 0s is less than we identify as a Unicode
             file (less than PercentageMatchUnicode) BUT a character greater
             than 127 was found, assume it is UTF8.
           - Else assume it's ASCII.
        Yes, technically speaking, the BOM is really only for identifying the
        byte order of the file but c'mon already... if your file isn't ASCII
        and you don't want it's encoding to be confused just put the BOM in
        there for pete's sake.
        Note: if you have a huge amount of text at the beginning of your file which
        is not code and is not single-byte, this algorithm may fail.  Again, put a 
        BOM in.
     #>
     $Content = (Get-Content -Path $Path -Encoding byte -ReadCount $ByteCountToCheck -TotalCount $ByteCountToCheck)
     $ByteCount = $Content.Count
     [bool]$NonAsciiFound = $false
 
     $ZeroCount = 0
     for ($i = 0; $i -lt $ByteCount; $i += 2) {
       if ($Content[$i] -eq 0) { $ZeroCount++ }
       if ($Content[$i] -gt 127) { $NonAsciiFound = $true }
     }
     if (($ZeroCount / ($ByteCount / 2)) -ge $PercentageMatchUnicode) {
       New-Object System.Text.UnicodeEncoding $true,$false
       return
     }
 
     $ZeroCount = 0
     for ($i = 1; $i -lt $ByteCount; $i += 2) {
       if ($Content[$i] -eq 0) { $ZeroCount++ }
       if ($Content[$i] -gt 127) { $NonAsciiFound = $true }
     }
     if (($ZeroCount / ($ByteCount / 2)) -ge $PercentageMatchUnicode) {
       New-Object System.Text.UnicodeEncoding $false,$false
       return
     }
 
     if ($NonAsciiFound -eq $true) {
       New-Object System.Text.UTF8Encoding $false
       return
     } else {
     [System.Text.Encoding]::"ASCII"
       return
     }
   }
 }
 Export-ModuleMember -Function Get-DTWFileEncoding
`

