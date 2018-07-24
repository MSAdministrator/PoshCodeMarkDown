---
Author: peter provost
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 696
Published Date: 2009-12-02t10
Archived Date: 2017-03-11t01
---

# elevate-process (sudo) - 

## Description

as a former unix guy, i love the non-admin stuff in vista, but got annoyed keeping two shells open (one admin and one non-admin). i wanted sudo! powershell made that easy. just put this in your $profile script. enjoy!

## Comments



## Usage



## TODO



## function

`elevate-process`

## Code

`#
 #
 function elevate-process
 {
 	$file, [string]$arguments = $args;
 	$psi = new-object System.Diagnostics.ProcessStartInfo $file;
 	$psi.Arguments = $arguments;
 	$psi.Verb = "runas";
 	$psi.WorkingDirectory = get-location;
 	[System.Diagnostics.Process]::Start($psi);
 }
 
 set-alias sudo elevate-process;
`

