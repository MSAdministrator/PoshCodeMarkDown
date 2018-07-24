---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.01
Encoding: ascii
License: cc0
PoshCode ID: 6101
Published Date: 2016-11-18t20
Archived Date: 2016-03-18t21
---

# get-filetail - 

## Description

an update to the orignal get-filetail that replaces a specific int with a long to accommodate particularly large files.  original text

## Comments



## Usage



## TODO



## script

`get-filetail`

## Code

`#
 #
 #.Synopsis 
 #.Description
 #.Parameter Name
 #.Parameter Lines
 #.Parameter Continuous
 #.Parameter Linesep
 #.Parameter Encoding
 #.Example
 #
 #.Example
 #
 
 PARAM( $Name, [int]$lines=10, [switch]$continuous, $linesep = "`n", [string]$encoding )
 BEGIN {
    if(Test-Path $Name) {
       $Name = (Convert-Path (Resolve-Path $Name))
    }
    [byte[]]$buffer = new-object byte[] 1024
 
    if($encoding) { 
       [System.Text.Encoding]$encoding = [System.Text.Encoding]::GetEncoding($encoding) 
       Write-Debug "Specified Encoding: $encoding"
    }
 
    function tailf {
    PARAM($StartOfTail=0)
       [string[]]$content = @()
       $reader = New-Object System.IO.FileStream $Name, "OpenOrCreate", "Read", "ReadWrite", 8, "None"
 
       if(!$encoding) {
          $b1 = $reader.ReadByte()
          $b2 = $reader.ReadByte()
          $b3 = $reader.ReadByte()
          $b4 = $reader.ReadByte()
 
          if (($b1 -eq 0xEF) -and ($b2 -eq 0xBB) -and ($b3 -eq 0xBF)) {
             Write-Debug "Detected Encoding:  UTF-8"
             [System.Text.Encoding]$encoding = [System.Text.Encoding]::UTF8
          } elseif (($b1 -eq 0) -and ($b2 -eq 0) -and ($b3 -eq 0xFE) -and ($b4 -eq 0xFF)) {
             Write-Debug "Detected Encoding:  12001 UTF-32 Big-Endian"
             [System.Text.Encoding]$encoding = [System.Text.Encoding]::GetEncoding(12001)
          } elseif (($b1 -eq 0xFF) -and ($b2 -eq 0xFE) -and ($b3 -eq 0) -and ($b4 -eq 0)) {
             Write-Debug "Detected Encoding:  12000 UTF-32 Little-Endian"
             [System.Text.Encoding]$encoding = [System.Text.Encoding]::UTF32
          } elseif (($b1 -eq 0xFE) -and ($b2 -eq 0xFF)) {
             Write-Debug "Detected Encoding:  1201 UTF-16 Big-Endian"
             [System.Text.Encoding]$encoding = [System.Text.Encoding]::BigEndianUnicode
          } elseif (($b1 -eq 0xFF) -and ($b2 -eq 0xFE)) {
             Write-Debug "Detected Encoding:  1200 UTF-16 Little-Endian"
             [System.Text.Encoding]$encoding = [System.Text.Encoding]::Unicode
          } elseif (($b1 -eq 0x2B) -and ($b2 -eq 0x2F) -and ($b3 -eq 0x76) -and (
                ($b4 -eq 0x38) -or ($b4 -eq 0x39) -or ($b4 -eq 0x2b) -or ($b4 -eq 0x2f))) {
             Write-Debug "Detected Encoding:  UTF-7"
             [System.Text.Encoding]$encoding = [System.Text.Encoding]::UTF7
          } else {
             Write-Debug "Unknown Encoding: [$('0x{0:X}' -f $b1) $('0x{0:X}' -f $b2) $('0x{0:X}' -f $b3) $('0x{0:X}' -f $b4)] using UTF-8"
             [System.Text.Encoding]$encoding = [System.Text.Encoding]::UTF8
          }
       }
       
       if($StartOfTail -eq 0) {
          $StartOfTail = $reader.Length - $buffer.Length
       } else {
          $OnlyShowNew = $true
       }
 
 
       do { 
          $pos = $reader.Seek($StartOfTail, "Begin")
          do {
             $count = $reader.Read($buffer, 0, $buffer.Length);
             $content += $encoding.GetString($buffer,0,$count).Split($linesep)
          } while( $count -gt 0 )
          $StartOfTail -= $buffer.Length
       } while(!$OnlyShowNew -and ($content.Length -lt $lines) -and $StartOfTail -gt 0)
 
       $end = $reader.Length
       
       if($content) {
          $output = [string]::Join( "`n", @($content[-$lines..-1]) )
          $len = $output.Length
          $output = $output.TrimEnd("`n")
          $end -= ($len - $output.Length) - 1
       }
 
       Write-Output $end
       if($output.Length -ge 1) {
          Write-Host $output -NoNewLine
       }
       $reader.Close();
    }
 }
 PROCESS {
    [long]$StartOfTail = tailf 0
 
    if($continuous) { 
       $Null = unregister-event "FileChanged" -ErrorAction 0
       $fsw = new-object system.io.filesystemwatcher
       $fsw.Path = split-path $Name
       $fsw.Filter = Split-Path $Name -Leaf
       $fsw.EnableRaisingEvents = $true
       $null = Register-ObjectEvent $fsw Changed "FileChanged" -MessageData $Name 
       while($true) {
          wait-event FileChanged | % { 
             [int]$StartOfTail = tailf $StartOfTail -newonly
             $null = Remove-Event $_.EventIdentifier
          }
       }
       unregister-event "FileChanged"
    }
    Write-Host
 }
 #}
`

