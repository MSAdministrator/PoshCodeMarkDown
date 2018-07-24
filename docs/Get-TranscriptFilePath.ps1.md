---
Author: oisin grehan
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2423
Published Date: 2011-12-27t08
Archived Date: 2015-07-24t21
---

# get-transcriptfilepath - 

## Description

get the name of the current (or last) transcription file used in the current session. requires powershell v2.0. some things to note

## Comments



## Usage



## TODO



## function

`get-transcriptfilepath`

## Code

`#
 #
 
 function Get-TranscriptFilePath {
     try {
       $externalHost = $host.gettype().getproperty("ExternalHost",
         [reflection.bindingflags]"nonpublic,instance").getvalue($host, @())
       $externalhost.gettype().getfield("transcriptFileName", "nonpublic,instance").getvalue($externalhost)
     } catch {
       write-warning "This host does not support transcription."
     }
 }
`

