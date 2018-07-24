---
Author: jaykul
Publisher: 
Copyright: 
Email: 
Version: 2.0
Encoding: ascii
License: cc0
PoshCode ID: 184
Published Date: 2008-04-22t21
Archived Date: 2016-05-19t06
---

# send-paste.ps1 - 

## Description

automated paste from powershell console

## Comments



## Usage



## TODO



## script

`send-paste`

## Code

`#
 #
 ##############################################################################################################
 ##############################################################################################################
 param( 
    $Title, 
    $Description="Automated paste from PowerShell console", 
    $KeepFor="d", 
    $Language="posh", 
    $url="http://posh.jaykul.com/p/" 
 )
    
 BEGIN {
    $null = [Reflection.Assembly]::LoadWithPartialName("System.Web")
    [string]$data = $null;
    [string]$meta = $null;
 
    if($language) {
       $meta = "format=" + [System.Web.HttpUtility]::UrlEncode($language)
    } else {
       $meta = "format=text"
    }
   
    do {
       switch -regex ($KeepFor) {
          "^d.*" { $meta += "&expiry=d" }
          "^m.*" { $meta += "&expiry=m" }
          "^f.*" { $meta += "&expiry=f" }
          default { 
             $KeepFor = Read-Host "Invalid value for 'KeepFor' parameter. Please specify 'day', 'month', or 'forever'"
          }
       }
    } until ( $meta -like "*&expiry*" )
 
    if($Description) {
       $meta += "&descrip=" + [System.Web.HttpUtility]::UrlEncode($Description)
    } else {
       $meta += "&descrip="
    }   
    $meta += "&poster=" + [System.Web.HttpUtility]::UrlEncode($Author)
    
    function PasteBin-Text ($meta, $title, $data) {
       $meta += "&paste=Send&posttitle=" + [System.Web.HttpUtility]::UrlEncode($Title)
       $data = $meta + "&code2=" + [System.Web.HttpUtility]::UrlEncode($data)
       
       
       $request = [System.Net.WebRequest]::Create($url)
       $request.ContentType = "application/x-www-form-urlencoded"
       $request.ContentLength = $data.Length
       $request.Method = "POST"
 
       $post = new-object IO.StreamWriter $request.GetRequestStream()
       $post.Write($data,0,$data.Length)
       $post.Flush()
       $post.Close()
 
       write-output $request.GetResponse().ResponseUri.AbsoluteUri
       $request.Abort()      
    }
 }
 
 PROCESS {
    switch($_) {
       {$_ -is [System.IO.FileInfo]} {
          $Title = $_.Name
          Write-Output $_.FullName
          Write-Output $(PasteBin-Text $meta $Title $([string]::join("`n",(Get-Content $_.FullName))))
       }
       {$_ -is [String]} {
          if(!$data -and !$Title){
             $Title = Read-Host "Give us a title for your post"
          }
          $data += "`n" + $_ 
       }
       default {
          Write-Error "Unable to process $_"
       }
    }
 }
 END {
    if($data) { 
       Write-Output $(PasteBin-Text $meta $Title $data)
    }
 }
 #}
`

