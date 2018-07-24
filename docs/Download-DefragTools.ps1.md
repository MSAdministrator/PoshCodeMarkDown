---
Author: carlos perez carlos_perez
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4045
Published Date: 2013-03-26t14
Archived Date: 2013-06-05t04
---

# download defragtools - 

## Description

downloads the all or the last video episode of defrag tools show from channel 9

## Comments



## Usage



## TODO



## script

`get-defragtoolsshow`

## Code

`#
 #
 <#
 .Synopsis
    Downloads Channel 9 Defrag Tool Episode Video
 .DESCRIPTION
    Downloads Channel 9 Defrag Tool Episode Video in the format selected and to a given path.
 .EXAMPLE
    Downloads all shows in WMV format to the default Downloads Folder for the user.
 
    Get-DefragToolsShow -All -VideoType wmv
 
 .EXAMPLE
    Downloads only the last show of the series in MP4 format
 
    Get-DefragToolsShow -Last -VideoType MP4
 .NOTES
     Author: Carlos Perez carlos_perez[at]darkoperator.com
 #>
 function Get-DefragToolsShow 
 {
     [CmdletBinding()]
     Param
     (
 
         [Parameter(Mandatory=$false,
                    Position=0)]
         $Path = "$($env:USERPROFILE)\downloads",
 
         [Parameter(Mandatory=$false,
         ParameterSetName="All")]
         [switch]$All = $true,
 
         [Parameter(Mandatory=$false,
         ParameterSetName="Last")]
         [switch]$Last = $true,
 
         [Parameter(Mandatory=$false)]
         [ValidateSet("MP4HD","MP4","WMVHD","WMV")]
         [string]$VideoType =  "MP4HD",
 
         [Parameter(Mandatory=$false,
         ParameterSetName="Last")]
         [switch]$Force = $true
         
     )
 
     Begin
     {
         $WebClient =  New-Object System.Net.WebClient
         $Global:downloadComplete = $false
         
         Unregister-Event -SourceIdentifier WebClient.DownloadProgressChanged -ErrorAction SilentlyContinue
         Unregister-Event -SourceIdentifier WebClient.DownloadFileComplete -ErrorAction SilentlyContinue
 
 
         Write-Verbose "Registering event for when tracking when download finishes."
         $eventDataComplete = Register-ObjectEvent $WebClient DownloadFileCompleted `
             -SourceIdentifier WebClient.DownloadFileComplete `
             -Action {$Global:downloadComplete = $true}
 
         Write-Verbose "Registering event for when tracking when download progress."
         $eventDataProgress = Register-ObjectEvent $WebClient DownloadProgressChanged `
             -SourceIdentifier WebClient.DownloadProgressChanged `
             -Action { $Global:DPCEventArgs = $EventArgs }    
 
         if (Test-Path $Path)
         {
             Set-Location (Resolve-Path $Path).Path
         }
         else
         {
             if ($Force)
             {
                 New-Item -ItemType directory -Path $Path | out-null
                 Set-Location (Resolve-Path $Path).Path
             }
             else
             {
                 Write-Error "Specified path does not exist"
                 return
             }
         }
     }
     Process
     {
         switch ($PsCmdlet.ParameterSetName)
         {
             "All"{$all = $true}
             "Last"{$all =  $false}
 
         }
 
         switch ($VideoType)
         {
             "MP4HD"  {$feedURL = "http://channel9.msdn.com/Shows/Defrag-Tools/feed/mp4high"} 
             "MP4"    {$feedURL = "http://channel9.msdn.com/Shows/Defrag-Tools/feed/mp4"}
             "WMVHD"  {$feedURL = "http://channel9.msdn.com/Shows/Defrag-Tools/feed/wmvhigh"}
             "WMV"    {$feedURL = "http://channel9.msdn.com/Shows/Defrag-Tools/feed/wmv"}
         }
 
         $feed = [xml]$WebClient.DownloadString($feedURL)
 
 
         if ($All)
         {
             foreach ($episode in $feed.rss.channel.Item)
             {
                 $episodeURL = [System.Uri]$episode.enclosure.url
 
                 $file = $episodeURL.Segments[-1]
                
                 if (!(Test-Path "$((Resolve-Path $Path).Path)\$($file)"))
                 {
                     Write-Progress -Activity 'Downloading file' -Status $file
                     $WebClient.DownloadFileAsync($episodeURL, "$((Resolve-Path $Path).Path)\$($file)")
 
                      while (!($Global:downloadComplete)) 
                      {                
                         $pc = $Global:DPCEventArgs.ProgressPercentage
                         if ($pc -ne $null) 
                         {
                             Write-Progress -Activity 'Downloading file' -Status $file -PercentComplete $pc
                         }
                     }
                     $Global:downloadComplete = $false
                 }
             }
         }
         else
         {
             $episodeURL = [System.Uri]$feed.rss.channel.Item[0].enclosure.url
             $file = $episodeURL.Segments[-1]
 
             if (!(Test-Path $file))
             {
                 Write-Progress -Activity 'Downloading file' -Status $file
                 $WebClient.DownloadFileAsync($episodeURL, "$((Resolve-Path $Path).Path)\$($file)")
                 while (!($Global:downloadComplete)) 
                 {                
                     $pc = $Global:DPCEventArgs.ProgressPercentage
                     if ($pc -ne $null) 
                     {
                         Write-Progress -Activity 'Downloading file' -Status $file -PercentComplete $pc
                     }
                 }
             }
         }
     }
     End
     {
         Unregister-Event -SourceIdentifier WebClient.DownloadProgressChanged
         Unregister-Event -SourceIdentifier WebClient.DownloadFileComplete
         $Global:downloadComplete = $null
         $Global:DPCEventArgs = $null
         [GC]::Collect()    
     }
 }
`

