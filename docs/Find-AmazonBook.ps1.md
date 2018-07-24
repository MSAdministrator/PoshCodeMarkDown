---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 725
Published Date: 
Archived Date: 2008-12-18t09
---

# find-amazonbook - 

## Description

find-amazonbook searches amazon’s books by title (and optionally author). it’s a simple example of the power of the httprest module, see the original here

## Comments



## Usage



## TODO



## module

`search-books`

## Code

`#
 #
 ####################################################################################################
 Set-HttpDefaultUrl "http://ecs.amazonaws.com/onca/xml"
 
 FUNCTION Search-Books {
 PARAM( [string]$Book, [string]$AWSKey, [string]$outFile, [string]$inputDelim = ",", [int]$Count = 1 )
 BEGIN {
    if($Book -and $(Test-Path $Book)) { 
       return Get-Content $Book | Search-Books 
    } elseif($Book){ $Book | Search-Books }
    
    $params = @{ 
       Service="AWSECommerceService"
       AWSAccessKeyId=$AWSKey
       Operation="ItemSearch"
       SearchIndex="Books"
    }
 }
 PROCESS { if($_){ $book=$_
    $bk = $book.Split($inputDelim)
    $params.Title = $bk[0].Trim()
    if($bk[1]) { $params.Author = $bk[1].Trim() } else { $params.Remove("Author") }
    
    $items = Invoke-Http GET -with $params | 
                Receive-Http Xml "//*[local-name() = 'Item']" |
                   Select Asin, DetailPageUrl,
                          @{n="Title";e={$_.ItemAttributes.Title}},
                          @{n="Author";e={$_.ItemAttributes.Author}} -First $Count
    if($outFile) {
       ConvertTo-CSV -io $items | Add-Content -Path $outFile
    } else {
       $items
    }
 }}
 }
`

