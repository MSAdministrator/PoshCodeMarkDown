---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 1.5
Encoding: ascii
License: cc0
PoshCode ID: 1463
Published Date: 
Archived Date: 2009-11-13t08
---

# convertto-relativetime - 

## Description

converts a pair of datetime’s or a timespan into a very approximate relative time like “about 4 days ago.”

## Comments



## Usage



## TODO



## function

`convertto-relativetimestring`

## Code

`#
 #
 function ConvertTo-RelativeTimeString {
 #.Synopsis 
 #.Description
 #.Parameter Span
 #.Parameter Before
 #.Parameter After
 #.Parameter Prefix
 #.Parameter Postfix
 [CmdletBinding(DefaultParameterSetName="TwoDates")]
 PARAM(
    [Parameter(ParameterSetName="TimeSpan",Mandatory=$true)]
    [TimeSpan]$span
 ,
    [Parameter(ParameterSetName="TwoDates",Mandatory=$true,ValueFromPipeline=$true)]
    [Alias("DateCreated")]
    [DateTime]$before
 ,
    [Parameter(ParameterSetName="TwoDates", Mandatory=$true, Position=0)]
    [DateTime]$after
 ,
    [Parameter(Position=1)]
    [String]$prefix = ""
 ,
    [Parameter(Position=2)]
    [String]$postfix = ""
  
 )
 PROCESS {
    if($PSCmdlet.ParameterSetName -eq "TwoDates") {
       $span = $after - $before
    }
    
    "$(
    switch($span.TotalSeconds) {
       {$_ -le 1}      { "$prefix a second $postfix "; break }     
       {$_ -le 60}     { "$prefix $($span.Seconds) seconds $postfix "; break }
       {$_ -le 120}    { "$prefix a minute $postfix "; break }
       default         { "$prefix $($span.Days) days $postfix "; break } 
    }
    )".Trim()
 }
 }
`

