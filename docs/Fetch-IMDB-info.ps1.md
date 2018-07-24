---
Author: dollarunderscore
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6822
Published Date: 2017-03-26t03
Archived Date: 2017-05-29t11
---

# fetch imdb info - 

## Description

i’m not sure if imdb updated their site or what, but much of the html code referenced in the script either no longer exists on the sites or doesn’t seem to grab the correct information.

## Comments



## Usage



## TODO



## function

`get-imdbmatch`

## Code

`#
 #
 function Get-IMDBMatch
 {
     <#
     .Synopsis
        Retrieves search results from IMDB
     .DESCRIPTION
        This cmdlet posts a search to IMDB and returns the results.
     .EXAMPLE
        Get-IMDBMatch -Title 'American Dad!'
     .EXAMPLE
        Get-IMDBMatch -Title 'American Dad!' | Where-Object { $_.Type -eq 'TV Series' }
     .PARAMETER Title
        Specify the name of the tv show/movie you want to search for.
 
     #>
 
     [cmdletbinding()]
     param([Parameter(Mandatory=$True, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
           [string[]] $Title)
 
     BEGIN { }
 
     PROCESS {
         foreach ($MediaTitle in $Title) {
             $IMDBSearch = Invoke-WebRequest -Uri "http://www.imdb.com/find?q=$($MediaTitle -replace " ","+")&s=all" -UseBasicParsing
 
             $FoundMatches = $IMDBSearch.Content -split "<tr class=`"findresult " | select -Skip 1 | % { (($_ -split "<TD class=`"result_text`">")[1] -split "</TD>")[0] } | Select-String -Pattern "fn_al_tt_"
 
             foreach ($Match in $FoundMatches) {
 
                 $ID = (($Match -split "/title/")[1] -split "/")[0]
                 $MatchTitle = (($Match -split ">")[1] -split "</a")[0]
                 $Released = (($Match -split "</a> \(")[1] -split "\)")[0]
                 $Type = (($Match -split "\) \(")[1] -split "\) ")[0]
 
                 if ($Type -eq "") {
                     $Type = "Movie"
                 }
 
                 if ($ID -eq "") {
                     Continue
                 }
 
                 $returnObject = New-Object System.Object
                 $returnObject | Add-Member -Type NoteProperty -Name ID -Value $ID
                 $returnObject | Add-Member -Type NoteProperty -Name Title -Value $MatchTitle
                 $returnObject | Add-Member -Type NoteProperty -Name Released -Value $Released
                 $returnObject | Add-Member -Type NoteProperty -Name Type -Value $Type
 
                 Write-Output $returnObject
 
                 Remove-Variable ID, MatchTitle, Released, Type
     
             }
 
             Remove-Variable FoundMatches, IMDBSearch
         }
     }
 
     END { }
 }
 
 
 function Get-IMDBItem
 {
     <#
     .Synopsis
        Retrieves information about a movie/tv show etc. from IMDB.
     .DESCRIPTION
        This cmdlet fetches information about the movie/tv show matching the specified ID from IMDB.
        The ID is often seen at the end of the URL at IMDB.
     .EXAMPLE
         Get-IMDBItem -ID tt0848228
     .EXAMPLE
        Get-IMDBMatch -Title 'American Dad!' | Get-IMDBItem
  
        This will fetch information about the item(s) piped from the Get-IMDBMatch cmdlet.
     .PARAMETER ID
        Specify the ID of the tv show/movie you want get. The ID has the format of tt0123456
     #>
  
     [cmdletbinding()]
     param([Parameter(Mandatory=$True, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
           [string[]] $ID)
  
     BEGIN { }
  
     PROCESS {
         foreach ($ImdbID in $ID) {
  
             $IMDBItem = Invoke-WebRequest -Uri "http://www.imdb.com/title/$ImdbID" -UseBasicParsing
  
             $ItemInfo = (($IMDBItem.Content -split "<div id=`"title-overview-widget`" class=`"heroic-overview`">")[1] -split "<div id=`"sidebar`">")[0]
  
             $ItemTitle = (($ItemInfo -split "<h1 itemprop=`"name`" class=`"`">")[1] -split "&nbsp;")[0]
             
             If (($ItemInfo -split "itemprop=`"datePublished`" content=`"").Length -gt 1) {
                 $Type = "Movie"
                 [DateTime]$Released = (($ItemInfo -split "<meta itemprop=`"datePublished`" content=`"")[1] -split "`" />")[0]
             } Else {
                 $Type = "TV Series"
                 $Released = $null
             }
  
             $Description = ((($ItemInfo -split "<div class=`"summary_text`" itemprop=`"description`">")[1] -split "</div>")[0]).Trim()
             
             $Rating = (($ItemInfo -split "<span itemprop=`"ratingValue`">")[1] -split "</span>")[0]
             
             $GenreSplit = $ItemInfo -split "itemprop=`"genre`">"
             $NumGenres = ($GenreSplit.Length)-1
             $Genres = foreach ($Genre in $GenreSplit[1..$NumGenres]) {
                 ($Genre -split "</span>")[0]
             }
 
             $MPAARating = (($ItemInfo -split "<meta itemprop=`"contentRating`" content=`"")[1] -split "`">")[0]
 
             try {
                 $RuntimeMinutes = New-TimeSpan -Minutes (($ItemInfo -split "<time itemprop=`"duration`" datetime=`"PT")[1] -split "M`">")[0]
             }
             catch {
                 $RuntimeMinutes = $null
             }
   
             if ($Description -like '*Add a plot*') {
                 $Description = $null
             }
  
             $returnObject = New-Object System.Object
             $returnObject | Add-Member -Type NoteProperty -Name ID -Value $ImdbID
             $returnObject | Add-Member -Type NoteProperty -Name Type -Value $Type
             $returnObject | Add-Member -Type NoteProperty -Name Title -Value $ItemTitle
             $returnObject | Add-Member -Type NoteProperty -Name Genre -Value $Genres
             $returnObject | Add-Member -Type NoteProperty -Name Description -Value $Description
             $returnObject | Add-Member -Type NoteProperty -Name Released -Value $Released
             $returnObject | Add-Member -Type NoteProperty -Name RuntimeMinutes -Value $RuntimeMinutes
             $returnObject | Add-Member -Type NoteProperty -Name Rating -Value $Rating
             $returnObject | Add-Member -Type NoteProperty -Name MPAARating -Value $MPAARating
  
             Write-Output $returnObject
  
             Remove-Variable IMDBItem, ItemInfo, ItemTitle, Genres, Description, Released, Type, Rating, RuntimeMinutes, MPAARating -ErrorAction SilentlyContinue
         }
     }
  
     END { }
 }
`

