---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1110
Published Date: 
Archived Date: 2009-05-19t06
---

# imaginaryfriendfeed - 

## Description

for friendfeed users

## Comments



## Usage



## TODO



## function

`new-imaginaryfriend`

## Code

`#
 #
 #.SYNOPSIS
 #.DESCRIPTION
 #.EXAMPLE
 #
 #
 
 function New-ImaginaryFriend {
 ##.Note
 ##.Synopsis
 ##.Example
 ##
 [CmdletBinding()]
 PARAM($name,[hashtable]$services,$avatar)
    if($global:ie -is [WatiN.Core.IE]) { 
       New-Watin "http`://friendfeed.com/settings/imaginary"
       while( $ie.Url -ne "http`://friendfeed.com/settings/imaginary" ) {
          Set-WatinUrl "http`://friendfeed.com/settings/imaginary"
          Read-Host "Press Enter after you have logged into FriendFeed"
       }
    }
    Set-WatinUrl http`://friendfeed.com/settings/imaginary
    if($ie.Url -ne "http`://friendfeed.com/settings/imaginary" ) {
       throw "You are not authenticated to FriendFeed. Please log in and try again"
    }
 
    Find-Button value "Create imaginary friend" | % { $_.Click() }
    Find-TextField name name | % { $_.TypeText( $name ) }
    (Find-Form action "/a/createimaginary").Buttons | % { $_.click() }
    
    $null = $ie.url -match "http`://friendfeed.com/([^/]*)/services";
    $ffid = $matches[1]
    
    foreach($service in $services.GetEnumerator()) {
       Write-Verbose "$($service.Key): $($service.Value)"
       Find-Link service $service.Key | % { $_.Click() }
       $form = Find-Form action "/a/configureservice"
       @($form.TextFields)[0].TypeText( $service.Value )
       @($form.Buttons)[0].click();
    }
    
    if($avatar) {
       Set-WatinUrl "http`://friendfeed.com/$ffid"
       $avatarFile = Get-WebFile $avatar ([uri]$avatar).segments[-1]
       Write-Verbose "Downloaded? $($avatarFile.FullName)"
       Start-Sleep -milli 500
       Find-Link innertext "Change picture" | %{ $_.Click() }
       Find-FileUpload id pictureupload | % { $_.Set( $avatarFile.FullName ) }
       (Find-Form action "/a/changepicture").Buttons | % { $_.click() }
       Start-Sleep -milli 500
       Remove-Item $avatarFile
    }
 }
 
 
 Function Get-FriendFeedFriends {
 Param([string]$UserName,[string[]]$Exclude)
    $friendNames = Get-WebPageText "http`://friendfeed.com/api/user/$username/profile" //user/subscription/nickname @{format="xml";include="subscriptions"} | 
                   Where {$Exclude -notcontains $_ }
 
    ForEach($incoming in (Get-WebPageContent "http`://friendfeed.com/api/profiles" //profiles/profile @{ format="xml"; nickname=$($friendNames -join ",") })) {
       $output = Select -Input $incoming Name, NickName | 
       Add-Member noteproperty Lists @($incoming.SelectNodes("//list/nickname").InnerText) -Passthru |
       Add-Member noteproperty Rooms @($incoming.SelectNodes("//room/nickname").InnerText) -Passthru
       ForEach($service in $incoming.service) {
          if(!$output."$($service.id)") {
             if($service.username) {
                Add-Member -input $output noteproperty $service.id $service.username
             } else {
                Add-Member -input $output noteproperty $service.id $service.profileUrl
             }
          } else {
             if($service.username) {
                $output."$($service.id)" = @($output."$($service.id)") + $service.username
             } else {
                $output."$($service.id)" = @($output."$($service.id)") + $service.profileUrl
             }
          }
       }
       $output
    }
 }
 
 function Get-TwitterFriends {
 Param($UserName, [string[]]$Exclude)
    $page = 0; $tweeters = @()
    do { 
       $tweeters = Get-WebPageContent "http`://twitter.com/statuses/friends.xml" //users/user @{page=(++$page); screen_name=$UserName} | 
                   Where {$Exclude -notcontains $_.screen_name }
       Write-Output $tweeters
    } while($tweeters)
 }
`

