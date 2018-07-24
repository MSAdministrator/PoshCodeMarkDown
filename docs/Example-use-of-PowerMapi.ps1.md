---
Author: powermapi
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6704
Published Date: 2017-01-20t00
Archived Date: 2017-01-24t08
---

# example use of powermapi - 

## Description

this shows a nice example/primer of using powermapi which is a module for powershell that provides direct access to mapi.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 $profile = "Outlook"
 $sess = new-MapiSession $profile
 
 $store = get-MapiStore -GetPrimaryStore
 
 $inbox = get-MapiFolder $store Inbox
 
 $found = search-MapiItems $inbox "PR_SUBJECT -has how to -or attach(PR_ATTACH_FILENAME -ew .zip) -or PR_BODY -match download{4,}"
 
 foreach ($item in $found) {
 	$mItem = get-MapiItem $inbox $item
 	export-MapiXML $mItem "c:\exported\folder\$($item.Subject).xml"
 	
 	remove-MapiItem $inbox $item
 }
 
 $inbox.MapiObject.Dispose()
 $store.MapiObject.Dispose()
 
 $pstStore = open-MapiPST $sess "c:\path\to\file.pst"
 
 $folder = get-MapiFolder $pstStore //stuff/2016
 
 $items = search-MapiItems $folder "PR_SUBJECT -has how to -or attach(PR_ATTACH_FILENAME -ew .zip) -or PR_BODY -match download{4,}"
 
 foreach ($item in $found) {
 	$mItem = get-MapiItem $inbox $item
 	export-MapiXML $mItem "c:\exported\folder\$($item.Subject).xml"
 	
 	remove-MapiItem $inbox $item
 }
 
 $folder.MapiObject.Dispose()
 $pstStore.MapiObject.Dispose()
 
 remove-MapiSession $sess
`

