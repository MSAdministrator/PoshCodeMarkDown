---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1931
Published Date: 2010-06-23t14
Archived Date: 2015-07-02t03
---

# copy-file (safely) - 

## Description

recursively copies all files and folders in $source folder to $destination folder, but with .copy inserted before the extension if the file already exists

## Comments



## Usage



## TODO



## function

`copy-file`

## Code

`#
 #
 function Copy-File {
 #.Synopsis
 param($source,$destination)
 
 mkdir $destination -force -erroraction SilentlyContinue
 
 foreach($original in ls $source -recurse) { 
   $result = $original.FullName.Replace($source,$destination)
   while(test-path $result -type leaf){ $result = [IO.Path]::ChangeExtension($result,"copy$([IO.Path]::GetExtension($result))") }
 
   if($original.PSIsContainer) { 
     mkdir $result -ErrorAction SilentlyContinue
   } else {
     copy $original.FullName -destination $result
   }
 }
 }
`

