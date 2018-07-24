---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.2
Encoding: ascii
License: cc0
PoshCode ID: 109
Published Date: 2008-01-11t16
Archived Date: 2012-07-12t04
---

# out-voice - 

## Description

speaks text using the sapi text-to-speech api

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 param([array]$Collection, [switch]$wait, [switch]$purge, [switch]$readfiles, [switch]$readxml, [switch]$notxml, [switch]$passthru)
 
 begin
 {  
   if ($args -eq '-?') {
 ''
 'Usage: Out-Speech [[-Collection] <array>]'
 ''
 'Parameters:'
 '    -Collection : A collection of items to speak.'
 '    -?          : Display this usage information'
 '  Switches:'
 '    -wait       : Wait for the machine to read each item (NOT asynchronous).'
 '    -purge      : Purge all other speech requests before making this call.'
 '    -readfiles  : Read the contents of the text files indicated.'
 '    -readxml    : Treat input as speech XML markup.'
 '    -notxml     : Do NOT parse XML (if text starts with "<" but is not XML).'
 '    -passthru   : Pass on the input as output.'
 ''
 'Examples:'
 '    PS> Out-Speech "Hello World"'
 '    PS> Select-RandomLine quotes.txt | Out-Speech -wait'
 '    PS> Out-Speech -readfiles "Hitchhiker''s Guide To The Galaxy.txt"'
 ''
       exit
   }
     
     #~ * Speak the given text string synchronously
     #~ * Not purge pending speak requests
     #~ * Parse the text as XML only if the first character is a left-angle-bracket (<)
     #~ * Not persist global XML state changes across speak calls
     #~ * Not expand punctuation characters into words.
   
   
   $SPF = $SPF_DEFAULT
   if(!$wait){ $SPF += $SPF_ASYNC }
   if($purge){ $SPF += $SPF_PURGEBEFORESPEAK }
   if($readfiles){ $SPF += $SPF_IS_FILENAME }
   if($readxml){ $SPF += $SPF_IS_XML }
   if($notxml){ $SPF += $SPF_IS_NOT_XML }
   
   $Voice = new-object -com SAPI.SpVoice
   
   if ($collection.count -gt 0) {
     foreach( $item in $collection ) {
       $exit = $Voice.Speak( ($item | out-string), $SPF )
     }
   }
 }
 
 process
 {
   if ($_)
   {
     $exit = $Voice.Speak( ($_ | out-string), $SPF )
     if($passthru) { $_ }
   }
 }
`

