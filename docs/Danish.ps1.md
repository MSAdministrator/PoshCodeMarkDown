---
Author: joel bennnett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 5232
Published Date: 2016-06-10t18
Archived Date: 2016-05-17t14
---

# danish - 

## Description

a script to use some web services for guessing languages and translating to english…

## Comments



## Usage



## TODO



## function

`resolve-language`

## Code

`#
 #
 ###################################################################################################
 ###################################################################################################
 ###################################################################################################
 ###################################################################################################
 [Reflection.Assembly]::LoadWithPartialName("System.Web") | Out-Null
 
 
 ###################################################################################################
 function Resolve-Language([string]$text) {
    return (ConvertFrom-HTML (Post-HTTP  "http://www.xrce.xerox.com/cgi-bin/mltt/LanguageGuesser" (
                                  "Text="+[System.Web.HttpUtility]::UrlEncode($text)))
 }
 
 ###################################################################################################
 function Get-English([string]$text,[string]$FromLanguage) {
    if(!$FromLanguage) {
       $FromLanguage = Resolve-Language $text
    }
    
    switch($FromLanguage) {
       "Arabic"       { $text = "langpair=ar|en&text=" + [System.Web.HttpUtility]::UrlEncode($text) }
       "Chinese"      { $text = "langpair=zh|en&text=" + [System.Web.HttpUtility]::UrlEncode($text) }
       "Dutch"        { $text = "langpair=nl|en&text=" + [System.Web.HttpUtility]::UrlEncode($text) }
       "French"       { $text = "langpair=fr|en&text=" + [System.Web.HttpUtility]::UrlEncode($text) }
       "German"       { $text = "langpair=de|en&text=" + [System.Web.HttpUtility]::UrlEncode($text) }
       "Greek"        { $text = "langpair=el|en&text=" + [System.Web.HttpUtility]::UrlEncode($text) }
       "Italian"      { $text = "langpair=it|en&text=" + [System.Web.HttpUtility]::UrlEncode($text) }
       "Japanese"     { $text = "langpair=ja|en&text=" + [System.Web.HttpUtility]::UrlEncode($text) }
       "Korean"       { $text = "langpair=ko|en&text=" + [System.Web.HttpUtility]::UrlEncode($text) }
       "Portuguese"   { $text = "langpair=pt|en&text=" + [System.Web.HttpUtility]::UrlEncode($text) }
       "Russian"      { $text = "langpair=ru|en&text=" + [System.Web.HttpUtility]::UrlEncode($text) }
       "Spanish"      { $text = "langpair=es|en&text=" + [System.Web.HttpUtility]::UrlEncode($text) }
       default { return "Sorry, but I can't translate $language" }
    }                                             
 }
 
 ###################################################################################################
 [regex]$anagram = "^Unscramble ... (.*)$"
 function Resolve-Anagram($anagram) {
    ((Post-HTTP "http://www.easypeasy.com/anagrams/results.php" "name=$anagram").Split("`n") |
     select-string "res 1" ) -replace ".*res 1.*value=""\s*([^""]*)\s*"".*",'$1'
 }
 
 ###################################################################################################
 function Post-HTTP($url,$bytes) {
    $request = [System.Net.WebRequest]::Create($url)
    $request.ContentType = "application/x-www-form-urlencoded"
    $request.ContentLength = $bytes.Length
    $request.Method = "POST"
    $rq = new-object IO.StreamWriter $request.GetRequestStream()
    $rq.Write($bytes)#,0,$bytes.Length)
    $rq.Flush()
    $rq.Close()
    $response = $request.GetResponse()
    $reader = new-object IO.StreamReader $response.GetResponseStream(),[Text.Encoding]::UTF8
    return $reader.ReadToEnd()
 }
`

