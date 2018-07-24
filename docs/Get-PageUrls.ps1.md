---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2159
Published Date: 2011-09-09t21
Archived Date: 2016-03-18t21
---

# get-pageurls.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 ##############################################################################
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Parse all of the URLs out of a given file.
 
 .EXAMPLE
 
 Get-PageUrls microsoft.html http://www.microsoft.com
 Gets all of the URLs from HTML stored in microsoft.html, and converts relative
 URLs to the domain of http://www.microsoft.com
 
 .EXAMPLE
 
 Get-PageUrls microsoft.html http://www.microsoft.com 'aspx$'
 Gets all of the URLs from HTML stored in microsoft.html, converts relative
 URLs to the domain of http://www.microsoft.com, and returns only URLs that end
 in 'aspx'.
 
 #>
 
 param(
     [Parameter(Mandatory = $true)]
     [string] $Path,
 
     [Parameter(Mandatory = $true)]
     [string] $BaseUrl,
 
     [switch] $Images,
     
     [string] $Pattern = ".*"
 )
 
 Set-StrictMode -Version Latest
 
 Add-Type -Assembly System.Web
 
 $regex = "<\s*a\s*[^>]*?href\s*=\s*[`"']*([^`"'>]+)[^>]*?>"
 if($Images)
 {
     $regex = "<\s*img\s*[^>]*?src\s*=\s*[`"']*([^`"'>]+)[^>]*?>"
 }
 
 function Main
 {
     $baseUrl = $baseUrl.Replace("\", "/")
 
     if($baseUrl.IndexOf("://") -lt 0)
     {
         throw "Please specify a base URL in the form of " +
             "http://server/path_to_file/file.html"
     }
 
     $baseUrl = $baseUrl.Substring(0, $baseUrl.LastIndexOf("/") + 1)
     $baseSlash = $baseUrl.IndexOf("/", $baseUrl.IndexOf("://") + 3)
 
     if($baseSlash -ge 0)
     {
         $domain = $baseUrl.Substring(0, $baseSlash)
     }
     else
     {
         $domain = $baseUrl
     }
 
 
     $content = [String]::Join(' ', (Get-Content $path))
     $contentMatches = @(GetMatches $content $regex)
 
     foreach($contentMatch in $contentMatches)
     {
         if(-not ($contentMatch -match $pattern)) { continue }
         if($contentMatch -match "javascript:") { continue }
 
         $contentMatch = $contentMatch.Replace("\", "/")
 
         if($contentMatch.IndexOf("://") -gt 0)
         {
             $url = $contentMatch
         }
         elseif($contentMatch[0] -eq "/")
         {
             $url = "$domain$contentMatch"
         }
         else
         {
             $url = "$baseUrl$contentMatch"
             $url = $url.Replace("/./", "/")
         }
 
         [System.Web.HttpUtility]::HtmlDecode($url)
     }
 }
 
 function GetMatches([string] $content, [string] $regex)
 {
     $returnMatches = new-object System.Collections.ArrayList
 
     $resultingMatches = [Regex]::Matches($content, $regex, "IgnoreCase")
     foreach($match in $resultingMatches)
     {
         $cleanedMatch = $match.Groups[1].Value.Trim()
         [void] $returnMatches.Add($cleanedMatch)
     }
 
     $returnMatches
 }
 
 . Main
`

