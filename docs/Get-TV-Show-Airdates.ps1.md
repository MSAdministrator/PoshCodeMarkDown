---
Author: dollarunderscore
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6551
Published Date: 2017-10-06t08
Archived Date: 2017-03-31t03
---

# get tv show airdates - 

## Description

these functions retrieve information about tv show airdates. they are used as a part of my “home automation with powershell”-project.

## Comments



## Usage



## TODO



## function

`get-tvshownextairdate`

## Code

`#
 #
 function Get-TVShowNextAirDate
 {
     <#
     .Synopsis
        Retrieves information about tv shows next air date
     .DESCRIPTION
        This cmdlet parses pogdesign's tv calendar for tv show air dates
     .EXAMPLE
        Get-TVShowNextAirDate
     .EXAMPLE
        Get-TVShowNextAirDate | Format-Table
     .EXAMPLE
        Get-TVShowNextAirDate | Where-Object { $_.TVShow -eq "Family Guy" }
     #>
 
     [CmdletBinding()]
     param()
 
     Begin {
         $URL = "http://www.pogdesign.co.uk/cat/next-airing"
         try {
             $CatTV = Invoke-WebRequest -Uri $URL -UseBasicParsing -ErrorAction Stop
         }
         catch {
             Write-Error "Failed to fetch TV Show data."
             return
         }
 
         $TVShows = $CatTV.Content -split "<div class=`"showlist" | Select-Object -Skip 1
     }
     Process {
         foreach ($TVShow in $TVShows) {
 
                 $TVShowName = ((($TVShow -split "<a href")[1] -split "</a>")[0] -split ">")[1] -replace "&amp;","&"
                 $EpisodeName = ((($TVShow -split "<a href")[2] -split "</a>")[0] -split ">")[1] -replace "&amp;","&"
                 $Season = (((((($TVShow -split "<a href")[2] -split "</a>")[0] -split ">")[0] -split "season-")[1]) -split "/Episode-")[0]
                 $Episode = (((((($TVShow -split "<a href")[2] -split "</a>")[0] -split ">")[0] -split "season-")[1]) -split "/Episode-")[1] -replace '"$'
                 $AirDateString = (($TVShow -split "epdate`">")[1] -split "</span>")[0]
                 $Year = "20" + (($AirDateString -split " ")[2] -replace "^'")
                 $MonthString = ($AirDateString -split " ")[1]
                 $Day = ($AirDateString -split " ")[0] -replace "[^\d]"
 
                 $Month = switch ($MonthString)
                                 {
                                     "Jan" {  "1" }
                                     "Feb" {  "2" }
                                     "Mar" {  "3" }
                                     "Apr" {  "4" }
                                     "May" {  "5" }
                                     "Jun" {  "6" }
                                     "Jul" {  "7" }
                                     "Aug" {  "8" }
                                     "Sep" {  "9" }
                                     "Oct" { "10" }
                                     "Nov" { "11" }
                                     "Dec" { "12" }
 
                                 }
                 $AirTime = ((($TVShow -split "eptime`">")[1] -split "</span>")[0] -split " ")[0]
                 $AirTimeAndDate = Get-Date "$Year-$Month-$Day $AirTime"
 
                 $returnObject = New-Object System.Object
                 $returnObject | Add-Member -Type NoteProperty -Name TVShow -Value $TVShowName
                 $returnObject | Add-Member -Type NoteProperty -Name EpisodeName -Value $EpisodeName
                 $returnObject | Add-Member -Type NoteProperty -Name Season -Value $Season
                 $returnObject | Add-Member -Type NoteProperty -Name Episode -Value $Episode
                 $returnObject | Add-Member -Type NoteProperty -Name AirDate -Value $AirTimeAndDate
 
                 Write-Output $returnObject
 
                 Remove-Variable TVShowName, EpisodeName, Season, Episode, AirTimeAndDate, AirDateString, Year, MonthString, Month, Day, AirTime, returnObject
         }
     }
 
     End { }
 }
 
 function Get-TVShowAirDate
 {
     <#
     .Synopsis
        Retrieves information about a specific tv show's air dates
     .DESCRIPTION
        This cmdlet parses pogdesign's tv calendar for a specific tv show's air dates
     .EXAMPLE
        Get-TVShowAirDate
     .EXAMPLE
        Get-TVShowAirDate | Format-Table
     .EXAMPLE
        Get-TVShowNextAirDate | Get-TVShowAirDate | ft
     #>
 
     [CmdletBinding()]
 
     param([Parameter(Mandatory=$True, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
           [string] $TVShow)
 
     Begin { }
 
     Process {
 
         $URL = "http://www.pogdesign.co.uk/cat/$($TVShow -replace ' ','-' -replace "&","and" -replace '[^a-zA-Z\d-]')-summary"
 
         try {
             $TVShowInfo = Invoke-WebRequest -Uri $URL -UseBasicParsing
         }
         catch {
             Write-Error "Failed to fetch TV Show data."
             return
         }
 
         #$TVShowEpisodeSpans = (($TVShowInfo.Content -split "Next Airing Episode Dates")[1] -split "Back to the TV Calendar")[0] -split "<div class=`"" | select -Skip 1
         $TVShowEpisodeSpans = ($TVShowInfo.Content -split '<li class="ep info "' | select -Skip 1) | %{ ($_ -split '<span class="punaired">UNAIRED</span>')[0] }
 
         foreach ($TVShowEpisodeSpan in $TVShowEpisodeSpans) {
             $EpisodeName = (($TVShowEpisodeSpan -split '</a></strong')[0] -split "`">")[5] -replace "&amp;","&"
             $Season = (($TVShowEpisodeSpan -split 'itemprop="seasonNumber" >')[1] -split "</span>")[0]
             $Episode = ((($TVShowEpisodeSpan -split 'itemprop="episodeNumber"')[1] -split "`">")[1] -split "</span>")[0]
             $AirDate = (($TVShowEpisodeSpan -split 'itemprop="releasedEvent" content="')[1] -split '"')[0]
             $AirTime = (((($TVShowEpisodeSpan -split 'itemprop="releasedEvent" content="')[1] -split '">')[1] -split "- ")[1] -split "</span>")[0]
             $AirDateTime = Get-Date "$AirDate $AirTime"
 
             $returnObject = New-Object System.Object
             $returnObject | Add-Member -Type NoteProperty -Name TVShow -Value $TVShow
             $returnObject | Add-Member -Type NoteProperty -Name EpisodeName -Value $EpisodeName
             $returnObject | Add-Member -Type NoteProperty -Name Season -Value $Season
             $returnObject | Add-Member -Type NoteProperty -Name Episode -Value $Episode
             $returnObject | Add-Member -Type NoteProperty -Name AirDate -Value $AirDateTime
 
             Write-Output $returnObject
 
             Remove-Variable EpisodeName, Season, Episode, AirDateTime
         }
     }
 
     End {  }
 }
`

