---
Author: _rov3
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 6314
Published Date: 2016-04-21t21
Archived Date: 2016-06-16t21
---

# mail sig gen xml - 

## Description

mail signature generation xml for https

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 <?xml version="1.0"?>
 <Settings>
 	<siglog>\\FOO\signatures_log.txt</siglog>
 	<siglogerror>\\FOO\signatures_errorlog.txt</siglogerror>
 	<searchbase>OU=FOO,DC=FOO,DC=FOO</searchbase>
 	<template>\\FOO\template.html</template>
 	<outputdir>\\FOO\\signatures</outputdir>
 	<modulepath>C:\Program Files\Mail Signature Generator\Scripts\createSignature.psm1</modulepath>
 	<tempdir>C:\Program Files\Mail Signature Generator\Temp</tempdir>
 	<generationinterval>3600</generationinterval>
 </Settings>
`

