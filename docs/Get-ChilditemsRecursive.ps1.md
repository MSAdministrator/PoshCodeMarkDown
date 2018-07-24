---
Author: bernd kriszio
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 999
Published Date: 2009-04-05t02
Archived Date: 2015-05-11t20
---

# get-childitemsrecursive - 

## Description

when you wish to compare to directory trees, you need the relative pathes of the files with respect to root of the directory tree, so that you can match them.

## Comments



## Usage



## TODO



## function

`get-childitem2`

## Code

`#
 #
 function Get-ChildItem2 ($path)
 {
     $root = gi $path
     $PathLength = $root.FullName.length
     gci $path -rec | % {
     Add-Member -InputObject $_ -MemberType NoteProperty -Name RelativePath -Value $_.FullName.substring($PathLength+1)
     $_
     }
 }
 
 
 cd f:
 md tmp
 md tmp\gcir\dir1
 md tmp\gcir\dir2
 "file1" > tmp\gcir\dir1\file1.txt
 "file2" > tmp\gcir\dir1\file2.txt
 
 Get-ChildItem2 tmp | % {$_.RelativePath }
 <#
 gcir
 gcir\dir1
 gcir\dir2
 gcir\dir1\file1.txt
 gcir\dir1\file2.txt
 gcir\dir2\file1.txt
 gcir\dir2\file2.txt
 #>
 
 
 Compare-Object (Get-ChildItem2 tmp\gcir\dir1) (Get-ChildItem2 tmp\gcir\dir2) -prop RelativePath, LastWriteTime, Length -includeEqual
 <#
 RelativePath        LastWriteTime                    Length SideIndicator      
 ------------        -------------                    ------ -------------      
 file1.txt           05.04.2009 10:45:03                  16 ==                 
 file2.txt           05.04.2009 10:45:03                  30 =>                 
 file2.txt           05.04.2009 10:45:03                  16 <=                 
 #>
 
 Compare-Object (Get-ChildItem2 tmp\gcir\dir1) (Get-ChildItem2 tmp\gcir\dir2) -prop RelativePath, LastWriteTime, Length
 
 $last = $null
 Compare-Object (Get-ChildItem2 tmp\gcir\dir1) (Get-ChildItem2 tmp\gcir\dir2) -prop RelativePath, LastWriteTime, Length |
 Sort RelativePath, LastWriteTime -desc | % {
 if ($last -ne $_.RelativePath)
 { $_ }
 $last = $_.RelativePath
 } | sort RelativePath
 
 <#
 RelativePath        LastWriteTime                    Length SideIndicator      
 ------------        -------------                    ------ -------------      
 file2.txt           05.04.2009 10:45:03                  30 =>                 
 #>
`

