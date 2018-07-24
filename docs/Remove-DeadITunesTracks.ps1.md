---
Author: mark schill
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 595
Published Date: 2009-09-20t23
Archived Date: 2015-03-14t01
---

# remove-deaditunestracks - 

## Description

this script will go through your itunes library and check the paths for each of the tracks. if it doesn’t find a file at the specified location it will remove that track from your itunes library.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Clear
 $ITunes = New-Object -ComObject iTunes.Application
 
 1..$ITunes.LibraryPlaylist.Tracks.Count | % {
 	$CurrentTrack = $ITunes.LibraryPlaylist.Tracks.Item($_)
 
 	if ( $CurrentTrack.Kind -eq 1 )
 		{
 		if ( ! ([System.IO.File]::Exists($CurrentTrack.Location)) ) 
 			{
 			Write-Host "$($CurrentTrack.Artist) - $($CurrentTrack.Name) has been deleted."
 			$CurrentTrack.Delete()
 			}
 		}
 	Write-Progress -activity "Removing Dead Tracks" -status "$($CurrentTrack.Artist) - $($CurrentTrack.Name)" -percentComplete ( $_/$ITunes.LibraryPlaylist.Tracks.Count*100)
 
 }
`

