---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6447
Published Date: 
Archived Date: 2016-07-18t18
---

#  - 

## Description

3 way file compare

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $folder1 = "C:\Users\Ciphr\Desktop\Test\738"
         $folder2 = "C:\Users\Ciphr\Desktop\Test\740"
 
 
         $firstFolder = Get-ChildItem -Recurse $folder1 | Where-Object { -not $_.PsIsContainer }
 
         $failedCount = 0
         $i = 0
         $totalCount = $firstFolder.Count
         $firstFolder | ForEach-Object {
             $i = $i + 1
             Write-Progress -Activity "Searching Files" -status "Searching File  $i of     $totalCount" -percentComplete ($i / $firstFolder.Count * 100)
             If ( Test-Path ( $_.FullName.Replace($folder1, $folder2) ) ) {
                 If ( Compare-Object (Get-Content $_.FullName) (Get-Content $_.FullName.Replace($folder1, $folder2) ) ) {
                     $fileSuffix = $_.FullName.TrimStart($folder1)
                     $failedCount = $failedCount + 1
                     Write-Host "$fileSuffix is on each server, but does not match"
                 }
             }
             else
             {
                 $fileSuffix = $_.FullName.TrimStart($folder1)
                 $failedCount = $failedCount + 1
                 Write-Host "$fileSuffix is only in folder 1"
 $fileSuffix | Out-File "$env:userprofile\desktop\Old version.txt" -Append
             }
         }
 
         $secondFolder = Get-ChildItem -Recurse $folder2 | Where-Object { -not $_.PsIsContainer }
 
         $i = 0
         $totalCount = $secondFolder.Count
         $secondFolder | ForEach-Object {
             $i = $i + 1
             Write-Progress -Activity "Searching for files only on second folder" -status "Searching File  $i of $totalCount" -percentComplete ($i / $secondFolder.Count * 100)
             If (!(Test-Path($_.FullName.Replace($folder2, $folder1))))
             {
                 $fileSuffix = $_.FullName.TrimStart($folder2)
                 $failedCount = $failedCount + 1
                 Write-Host "$fileSuffix is only in folder 2"
 $fileSuffix | Out-File "$env:userprofile\desktop\folder2.txt" -Append
             }
         }
`

