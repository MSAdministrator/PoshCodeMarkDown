---
Author: jan egil ring
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 3399
Published Date: 2012-05-06t04
Archived Date: 2012-05-13t23
---

# get-webvideofile - 

## Description

&#65279;download video-files from the specified rss-feed url, based on html scraping and a regular expression for finding the download url.joel bennett`s get-webfile function from poshcode.org, which provides progress status during download, is used for downloading the files.

## Comments



## Usage



## TODO



## script

`get-webfile`

## Code

`#
 #
 <#
 .SYNOPSIS
 Download video-files from the specified RSS-feed URL, based on HTML scraping and a regular expression.
 
 .DESCRIPTION
 Download video-files from the specified RSS-feed URL, based on HTML scraping and a regular expression for finding the download URL.
 Joel Bennett`s Get-WebFile function from poshcode.org, which provides progress status during download, is used for downloading the files.
 The script was originally created for downloading wmv-files from Microsoft TechNet Edge (http://technet.microsoft.com/en-us/edge).
 
 .PARAMETER RssUrl 
 The URL for the RSS feed to process
 
 .PARAMETER destination
 The destination-folder for the downloaded video files. If not specified, the downloaded files will be placed in the current user`s Video-folder ($home\Videos).
 
 .PARAMETER UseOriginalFileName
 Switch-parameter to specify usage of original filenames. If not specified the RSS title will be used as filename.
 
 .PARAMETER UrlRegex
 A regular expression used to search for video URL`s. If not specified a regular expression for finding wmv-files on TechNet Edge is used.
 
 .EXAMPLE
 .\Get-WebVideoFile.ps1 -RssUrl "http://technet.microsoft.com/en-us/edge/SyndicationGetTopics/cc543196.aspx?field=Category&value=System Center 2012&ancestor=ff524487&version=MSDN.10"
 
 .EXAMPLE
 .\Get-WebVideoFile.ps1 -Destination "C:\TechNet Edge Videos\" -RssUrl "http://technet.microsoft.com/en-us/edge/SyndicationGetTopics/cc543196.aspx?field=Category&value=System Center 2012&ancestor=ff524487&version=MSDN.10"
 
 .EXAMPLE
 .\Get-WebVideoFile.ps1 -UseOriginalFileName -RssUrl "http://technet.microsoft.com/en-us/edge/SyndicationGetTopics/cc543196.aspx?field=Category&value=System Center 2012&ancestor=ff524487&version=MSDN.10"
 
 .EXAMPLE
 .\Get-WebVideoFile.ps1 -Verbose -RssUrl "http://technet.microsoft.com/en-us/edge/SyndicationGetTopics/cc543196.aspx?field=Category&value=System Center 2012&ancestor=ff524487&version=MSDN.10"
 
 
 .NOTES
 
  Name: Get-WebVideoFile.ps1
  Author: Jan Egil Ring
  Website: http://blog.powershell.no
 
  Usage:
  1) Find and browse to the category you want to download files from. Available categories: http://technet.microsoft.com/en-us/edge/ff701756
  2) Find the RSS URL by clicking the RSS-icon next to the category title on the top of the website
  3) Specify the URL on the URL-parameter: .\Get-WebVideoFile.ps1 -RssUrl "http://technet.microsoft.com/en-us/edge/Syndication..."
 
  You have a royalty-free right to use, modify, reproduce, and
  distribute this script file in any way you find useful, provided that
  you agree that the creator, owner above has no warranty, obligations,
  or liability for such use.
 
  VERSION HISTORY:
  1.0 05.05.2012 - Initial release
 
 #>
 
 
 Param(
        [Parameter(Mandatory=$true)]
 	   [string]$RssUrl,
        [string]$Destination = "$home\Videos\",
        [switch]$UseOriginalFileName,
        [regex]$UrlRegex = "(?<url>http://content\d.catalog.video.msn.com/../../[0-f]{8}-[0-f]{4}-[0-f]{4}-[0-f]{4}-[0-f]{12}(?<file>[^>]*?wmv))"
 )
 
 
 function Get-WebFile {
    param( 
       $url = (Read-Host "The URL to download"),
       $fileName = $null,
       [switch]$Passthru,
       [switch]$quiet
    )
    
    if($url.contains("http"))
    {
    $req = [System.Net.HttpWebRequest]::Create($url);
    }
    else
    {
    $URL_Format_Error = [string]"Connection protocol not specified. Recommended action: Try again using protocol (for example 'http://" + $url + "') instead. Function aborting...";
    Write-Error $URL_Format_Error;
    return;
    }
    
    $req.CookieContainer = New-Object System.Net.CookieContainer
 
    try{
    $res = $req.GetResponse();
    }
    catch
    {
    Write-Error $error[0].Exception.InnerException.Message;
    return;
    }
  
    if($fileName -and !(Split-Path $fileName)) {
       $fileName = Join-Path (Get-Location -PSProvider "FileSystem") $fileName
    } 
    elseif((!$Passthru -and ($fileName -eq $null)) -or (($fileName -ne $null) -and (Test-Path -PathType "Container" $fileName)))
    {
       [string]$fileName = ([regex]'(?i)filename=(.*)$').Match( $res.Headers["Content-Disposition"] ).Groups[1].Value
       $fileName = $fileName.trim("\/""'")
       if(!$fileName) {
          $fileName = $res.ResponseUri.Segments[-1]
          $fileName = $fileName.trim("\/")
          if(!$fileName) { 
             $fileName = Read-Host "Please provide a file name"
          }
          $fileName = $fileName.trim("\/")
          if(!([IO.FileInfo]$fileName).Extension) {
             $fileName = $fileName + "." + $res.ContentType.Split(";")[0].Split("/")[1]
          }
       }
       $fileName = Join-Path (Get-Location -PSProvider "FileSystem") $fileName
    }
    if($Passthru) {
       $encoding = [System.Text.Encoding]::GetEncoding( $res.CharacterSet )
       [string]$output = ""
    }
  
    if($res.StatusCode -eq 200) {
       [long]$goal = $res.ContentLength
       $reader = $res.GetResponseStream()
       if($fileName) {
          try{
          $writer = new-object System.IO.FileStream $fileName, "Create"
          }
          catch{
          Write-Error $error[0].Exception.InnerException.Message;
          return;
          }
       }
       [byte[]]$buffer = new-object byte[] 4096
       [long]$total = [long]$count = 0
       do
       {
          $count = $reader.Read($buffer, 0, $buffer.Length);
          if($fileName) {
             $writer.Write($buffer, 0, $count);
          } 
          if($Passthru){
             $output += $encoding.GetString($buffer,0,$count)
          } elseif(!$quiet) {
             $total += $count
             if($goal -gt 0) {
                Write-Progress "Downloading $url" "Saving $total of $goal" -id 0 -percentComplete (($total/$goal)*100)
             } else {
                Write-Progress "Downloading $url" "Saving $total bytes..." -id 0
             }
          }
       } while ($count -gt 0)
       
       $reader.Close()
       if($fileName) {
          $writer.Flush()
          $writer.Close()
       }
       if($Passthru){
          $output
       }
    }
    $res.Close(); 
    if($fileName) {
       ls $fileName
    }
 }
 
 
 $wc = New-Object net.webclient
 [xml]$xml = $wc.DownloadString($rssurl)
 $itemcount = $xml.rss.channel.item.count
 $count = 0
 
 $xml.rss.channel.item | foreach {
 
 $count ++
 
 Write-Verbose "Processing RSS item $count of $itemcount : $($_.title)"
 
 $string = $wc.DownloadString($_.link)
 
     if ($string -match $urlregex) {
     
     Write-Verbose "URL regex matched"
     
         $url = $matches.url        
     }
     else {
     
     Write-Verbose "URL regex did not match"
     
     return
 
     }
 
     if ($UseOriginalFileName) {
 
          $file = $url.split("/")[-1]
 
          }
       
       else {
 
          $file = $_.Title
 
          foreach ($character in ('/','?','*',':',';','{','}','\','|')) {
          $file = $file.Replace($character,'')
          }
 
          $file = $file + '.' + $url.split(".")[-1]
          }
 
 if ($url) {
 
 $filepath =  "$destination$file"
         if (Test-Path $filepath) 
         {Write-Verbose "$file is already present"}
         else {
             Write-Verbose "Downloading $file"
 
 
 Get-WebFile -url $url -filename $filepath
 
 
         }    
     }
 }
`

