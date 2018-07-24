---
Author: beefarino
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 3159
Published Date: 2012-01-09t11
Archived Date: 2012-01-15t04
---

# out-playlist.ps1 - 

## Description

aaron nelson asked via twitter

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 param( 
 	[parameter(Mandatory=$true)]
 	[string]
 	$name,
 	
 	[parameter(Mandatory=$true,ValueFromPipeline=$true)]
 	$file
 )
 
 begin
 {
 	$script:files = @();
 }
 process
 {
 	$script:files += $file.fullname;
 }
 end
 {
 	$count = $script:files.length;
 	$mediaElements = $script:files | foreach{
 		"<media src='$_' />"
 	};
 	
 [xml]$playlist = @"
 <?wpl version="1.0"?>
 <smil>
     <head>
         <meta name="Generator" content="out-playlist.ps1"/>
         <meta name="IsNetworkFeed" content="0"/>
         <meta name="ItemCount" content="$count"/>
         <title>$name</title>
     </head>
     <body>
         <seq>
 			$mediaElements
         </seq>
     </body>
 </smil>
 "@;
 	new-item "~\documents\my music\playlists\$name.wpl" -value '' -type file -force | out-null;
 	$playlist.save( ("~\documents\my music\playlists\$name.wpl" | resolve-path) );
 }
`

