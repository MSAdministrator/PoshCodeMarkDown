---
Author: jpbruckler
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5524
Published Date: 2014-10-18t17
Archived Date: 2014-10-20t18
---

# write-console - 

## Description

writes a message to the console with colored foreground and background. given messages are split on word boundaries into lines, and each line is padded to $limit length in order to allow the background color of the message to extend to $limit consistently.

## Comments



## Usage



## TODO



## function

`write-console`

## Code

`#
 #
 Function Write-Console
 {
   <#
     .SYNOPSIS
       Writes colored and wrapped messages to the console using Write-Host
     .DESCRIPTION
       Outputs a given message to the console host wrapped at a given character
       count. The wrapping behavior splits at word boundaries to create lines.
       Each line is then padded to the count provided by the Limit parameter so
       that the colored output is consistent length.
     .PARAMETER Message
       [string] The message to write to the console.
     .PARAMETER Limit
       [int] The maximum string length for any given line of text.
     .PARAMETER Type
       [string] Defines what type of message to output (affects coloring).
       Defaults to Success.
       Possible values:
         Error - Dark red background, light red text
         Success - Dark green background, light green text
         Warning - Black background, yellow text
     .INPUTS
       [string] The message to be printed to the screen.
     .OUTPUTS
       None.
     .EXAMPLE
       PS C:\> Write-Console -Message 'This is a really long message that should be split at 30 characters.' -Limit 30
   #>
   param(
     [Parameter(Mandatory=$true)]
     [string] $Message,
     [int] $Limit = 72,
     [ValidateSet('Error','Warning','Success')]
     [string] $Type = 'Success'
   )
 
   begin
   {
     $SB = New-Object System.Text.StringBuilder
     $Lines = New-Object System.Collections.ArrayList
     $Words = New-Object System.Collections.ArrayList
   }
 
   process
   {
     switch ($Type)
     {
       'Success' { $FG = 'Green'; $BG = 'DarkGreen' }
       'Error'   { $FG = 'Red'; $BG = 'DarkRed' }
       'Warning' { $FG = 'Yellow'; $BG = 'Black'}
     }
     
     $Message -split ' ' | ForEach-Object {
       if ($PSItem.Length -gt $Limit)
       {
         $Split = [math]::Ceiling($PSItem.Length/$Limit)
         $Parts = [regex]::Split($PSitem,$Split,$Limit)
         $Split
         foreach($Part in $Parts)
         {
           $Part
           $null = $Words.Add($Part)
         }
       }
       else
       {
         $null = $Words.Add($PSItem)
       }
     }
     
     $c = $Words.Count
     for ($i=0;$i -le $c;$i++)
     {     
       if ($i -eq $c)
       {
         $Line = $SB.ToString().PadRight($Limit,' ')
         $null = $SB.Clear()
         $null = $Lines.Add($Line)
         $null = $SB.Append("$($Words[$i]) ")
         break;
       }
 
       if ($SB.Length + ($Words[$i].Length + 1) -le $Limit)
       {
         $null = $SB.Append("$($Words[$i]) ")
       }
       else
       {
         $Line = $SB.ToString().PadRight($Limit,' ')
         $null = $SB.Clear()
         $null = $Lines.Add($Line)
         $null = $SB.Append("$($Words[$i]) ")
       }
 
     }
 
     foreach ($Line in $Lines)
     {
       Write-Host $Line -ForegroundColor $FG -BackgroundColor $BG
     }
   }
 
   end
   {
     $SB,$Lines,$Words = $null
   }
 }
`

