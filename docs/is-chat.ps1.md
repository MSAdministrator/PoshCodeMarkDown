---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4506
Published Date: 
Archived Date: 2013-10-11t21
---

#  - 

## Description

twitch chat image scraper.

## Comments



## Usage



## TODO



## script

`is-chat`

## Code

`#
 #
 #
 #(note: I like dot-including the script over &-running it so I can mess with the $chatObjects collection afterwards
 
 
 param($channelName, [System.IO.FileInfo]$optionalChatLogFullPathAndFilename = $null,[datetime]$startTimeUtc)
 
 if ($optionalChatLogFullPathAndFilename -ne $null) {
   $logFile = $optionalChatLogFullPathAndFilename
 } else {
   $logFile = @(dir -path (join-path $env:AppData "HexChat\logs") -recurse -include "#$($channelname).log") | select -first 1
 }
 Write-Host "Scanning $($logFile.fullname)"
 
 $bannedUsers = @(
   "packattack91", 
   "drypaperbag",
   "bunnyhro"
 )
 
 $boilerPlate = @"
 <html>
 <head>
 <style>
 body {{
   font-family: Helvetica, Arial, sans-serif;
 }}
 table tr td {{
   padding: 2px;
 }}
 td.timestamp {{
   width: 190px;
 }}
 tr.alternate-row {{
 }}
 </style>
 </head>
 <body>
 <table border=1>
 
 {0}
 
 </table>
 </body>
 </html>
 "@
 
  function is-chat($line) {
    $isFromABannedUser = $false
    $bannedUsers | foreach {
      if ($line -like "*<$($_)>*") {
        $isFromABannedUser = $true
      }
    }
    return (-not $isFromABannedUser) -and [regex]::IsMatch($line, '<.*?>') -and ($line -notlike "*vine*4*u*")
  }
 
  
  function get-chatline($line) {
    $o = new-object psobject
    add-member -membertype "NoteProperty"  -name "Timestamp" -inputobject $o -value (process-timestamp -timestamp ($line.Substring(0,15)))
    add-member -membertype "NoteProperty"  -name "TimeSinceStart" -inputobject $o -value (Get-Offset -timestamp $o.Timestamp)
 
    add-member -membertype "NoteProperty"  -name "Name" -inputobject $o -value (get-name -line $line)
    add-member -membertype "NoteProperty"  -name "Text" -inputobject $o -value (get-text -line $line)
    
    add-member -membertype "NoteProperty"  -name "ImageLink" -inputobject $o -value (get-imagelink -line $line)
    return $o
  }
  
  
 function get-imagelink($line) {
   $imageRegex = '(http\S+?(jpeg|jpg|png|gif))|(http://imgur.com/\S+)'
   if ($line -match $imageRegex) {
     return [regex]::Match($line, $imageRegex).Value
   } else { 
     return $null
   }
 }
 
 [reflection.assembly]::LoadWIthPartialName("System.Web") | out-null
 $script:alternateRow = $false
 
 function format-html($o, [switch]$makeImgTags) {
   if ($script:alternateRow -eq $true) {
     $rowClass = "standard-row"
   } else {
     $rowClass = "alternate-row"
   }
   
   $script:alternateRow = -not $script:alternateRow
   
   $additionalTextContent = ""
   if ($makeImgTags -eq $true) {
     $additionalTextContent = "<br /><img src=""" + $o.ImageLink + """ /><br /><a href=""" + $o.ImageLink + """>$($o.ImageLink)</a>"
   }
   
   return @"
 <tr class="$rowClass">
 <td class="timestamp">$($o.Timestamp.ToString())</td>
 <td class="time-since-start">$($o.TimeSinceStart)</td>
 <td class="name">$($o.Name)</td>
 <td class="text">$([System.Web.HttpUtility]::HtmlEncode($o.Text))$($additionalTextContent)</td>
 </tr>
 "@
 
 }
 
 function get-offset([datetime]$timestamp) {
   if ($timeStamp -lt $startTimeUtc) {
     return ""
   } else {
     $diff = $timestamp - $startTimeUtc
     if ($diff.Hours -gt 0 -or $diff.Days -gt 0) {
       $hoursPart = "$($diff.Days*24 + $diff.Hours)h"
     } else {
       $hoursPart = ""
     }
     return "$($hoursPart)$($diff.Minutes)m$($diff.Seconds)s"
   }
 }
 
 function get-text($line) {
   [Regex]::Match($line, '<.*?>.(?<text>.*?)$').Groups['text'].Value
 }
 
 function get-name($line) {
   [Regex]::Match($line, '<(?<name>.*?)>').Groups['name'].Value
 }
 
  
  function process-timestamp($timestamp) {
    $year = [DateTime]::Today.Year.ToString()
    $monthString = $timestamp.Substring(0,3)
    $month = [DateTime]::Parse("1.$($monthString) 2010").Month.ToString()
    $day = $timestamp.Substring(4,2)
    $hour = $timestamp.Substring(7,2)
    $minute = $timestamp.Substring(10,2)
    $second = $timestamp.Substring(13,2)
    
    ([DateTime]"$year-$month-$day $($hour):$($minute):$($second)").ToUniversalTime()
  }
 
 function Make-HtmlChatLog($chatObjects, $filename, [switch]$makeImgTags) {
   $chatTableRows = $chatObjects | foreach { format-html $_ -makeImgTags $makeImgTags }
   $allHtml = [string]::Format($boilerPlate, [string]::Join("`n", $chatTableRows))
   $allHtml > $filename
   ii $filename
 }
 
 function format-column($content, [int]$length, [string]$fillCharacter) {
   $spacesToAdd = [math]::Max(0,$length - $content.Length)
   return "|" + $content + ($fillCharacter * $spacesToAdd) 
 }
 
 function format-redditrow($o, [switch]$useDashesForFillCharacter) {
   $fillCharacter = " "
   if ($useDashesForFillCharacter -eq $true) {
     $fillCharacter = "-"
   }
   
   $line = format-column -content ($o.TimeStamp.ToString()) -length 22 -fillCharacter $fillCharacter
   $line += format-column -content $o.TimeSinceStart -length 10 -fillCharacter $fillCharacter
   $line += format-column -content $o.Name -length 20 -fillCharacter $fillCharacter
   $line += format-column -content $o.Text -length 100 -fillCharacter $fillCharacter
   return $line
 }
 
 function Make-RedditTable($chatObjects, $filename) {
   $header = @{"Timestamp" = "Timestamp (UTC)"; "TimeSinceStart" = "~offset"; "Name" = "twitch chat name"; "Text" = "text"}
   $dividerLineObject = @{"Timestamp" = ":"; "TimeSinceStart" = ":"; "Name" = ":"; "Text" = ":"}
   $chatTableHeader = format-redditRow $header
   $dividerLine = "`r`n" + (format-redditRow -o $dividerLineObject -useDashesForFillCharacter) + "`r`n"
   $chatTableRows = $chatObjects | foreach { format-RedditRow $_ -makeImgTags $makeImgTags }
   $allText = $chatTableHeader + $dividerLine + ([string]::Join("`r`n", $chatTableRows))
   $allText > $filename
   ii $filename
 }
 
 function Does-NoBetterImageExist($image, $fullList) {
   $betterImage = $fullList | select -expand ImageLink | where { $_ -like "$($image.ImageLink)*" -and $_ -ne $image.ImageLink }
   if ($betterImage -ne $null) {
     return $false
   } else {
     return $true
   }
 }
 
 $chatLog = cat $logFile
 $chatObjects = $chatlog | where { is-chat $_ } | foreach { get-chatline $_ }
 
 $imageLinks = $chatObjects | where { $_.imagelink -ne $null } | group imagelink | foreach { $_.group | sort timestamp | select -first 1 } | sort timestamp
 $curatedImageLinks = $imageLinks | where { Does-NoBetterImageExist -image $_ -fullList $imageLinks }
 
 Make-RedditTable -chatObjects $curatedImageLinks -filename "$($channelName)-linked-images.txt"
 Make-htmlChatLog -chatObjects $curatedImageLinks -filename "$($channelName)-linked-images-preview.html" -makeImgTags
`

