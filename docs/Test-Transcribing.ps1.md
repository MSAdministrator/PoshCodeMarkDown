---
Author: oisin grehan
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1500
Published Date: 2010-12-01t09
Archived Date: 2017-05-22t03
---

# test-transcribing - 

## Description

start-transcript and stop-transcript will start and stop a file-based transcription log. however, there is no way to tell (afaik) if the current host is actually transcribing. test-transcribing will return $true if we are transcribing, $false if not.

## Comments



## Usage



## TODO



## function

`test-transcribing`

## Code

`#
 #
 
 function Test-Transcribing {
 	$externalHost = $host.gettype().getproperty("ExternalHost",
 		[reflection.bindingflags]"NonPublic,Instance").getvalue($host, @())
 
 	try {
 	    $externalHost.gettype().getproperty("IsTranscribing",
 		[reflection.bindingflags]"NonPublic,Instance").getvalue($externalHost, @())
 	} catch {
              write-warning "This host does not support transcription."
          }
 }
`

