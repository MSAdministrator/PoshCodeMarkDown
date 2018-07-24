---
Author: james gentile
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 3328
Published Date: 2012-04-08t15
Archived Date: 2012-09-29t02
---

# finddupe.ps1 - 

## Description

find duplicate files. this version is orders of magnitude faster than the previous.

## Comments



## Usage



## TODO



## function

`get-sha512`

## Code

`#
 #
 function Get-SHA512([System.IO.FileInfo] $file = $(throw 'Usage: Get-MD5 [System.IO.FileInfo]'))
 {
   	$stream = $null
   	$cryptoServiceProvider = [System.Security.Cryptography.SHA512CryptoServiceProvider]
   	$hashAlgorithm = new-object $cryptoServiceProvider
   	try 
     {
       $stream = $file.OpenRead()
   	}
   	catch { return $null }
   	$hashByteArray = $hashAlgorithm.ComputeHash($stream)
   	$stream.Close()
   	trap
   	{
    		if ($stream -ne $null)
     	{
         $stream.Close()
       }
       return $null
     } 	
     foreach ($byte in $hashByteArray) { if ($byte -lt 16) {$result += �0{0:X}� -f $byte } else { $result += �{0:X}� -f $byte }}
     return [string]$result
 }
 
 $starttime=[datetime]::now
 write-host "FindDupe.ps1 - find and optionally delete duplicate files. FindDupe.ps1 -help or FindDupe.ps1 -h for usage options."
 $m = 0
 $args3=$args
 $args2=$args3|?{$_ -ne "-delete" -and $_ -ne "-recurse" -and $_ -ne "-hidden" -and $_ -ne "-h" -and $_ -ne "-help"}
 if ($args3 -eq "-help" -or $args3 -eq "-h")
 {
 	""
 	"Usage:"
 	"Options:"
 	"       -recurse recurses through all subdirectories of any specified directories."
 	"	      -hidden checks hidden files, default is to ignore hidden files."
 	"	      -help displays this usage option data, and ignores all other arguments."
 	""
 	"Examples:"
 	"          PS>.\finddupe.ps1 c:\data d:\finance -recurse"
 	"          PS>.\finddupe.ps1 d: -recurse -delete"
 	"          PS>.\finddupe.ps1 c:\users\alice\pictures\ -recurse -delete"
  	exit
 }
 
 
 $files=@(dir -ea 0 $args2 -recurse:$([bool]($args3 -eq "-recurse")) -force:$([bool]($args3 -eq "-hidden")) |?{$_.psiscontainer -eq $false}|sort length) 
 if ($files.count -lt 2) {exit}
 $sizenamehash=@{}
 
 for ($i=0;$i -lt ($files.count-1); $i++)
 {  
   if ($files[$i].length -ne $files[$i+1].length) {continue}
   $breakout=$false
   while($true)
   {    
     $sha512 = (get-SHA512 $files[$i].fullname)
     if ($sha512 -ne $null)
     {
       if (($sizenamehash.$($files[$i].length)) -ne $null)
       {            
         if ($sizenamehash.$($files[$i].length).$($files[$i].fullname) -eq $null)
         {
           $sizenamehash.$($files[$i].length)+=@{$($files[$i].fullname)=$sha512}      
         }
       }              
       else
       {
         $sizenamehash+=@{$($files[$i].length)=@{$($files[$i].fullname)=$sha512}}
       }
     }
     if ($breakout -eq $true) {break}  
     $i++    
     if ($i -eq ($files.count-1)) {$breakout=$true; continue}
     $breakout=(($files[$i].length -ne $files[$i+1].length))    
   }    
 } 
 
 ($sizenamehash.getenumerator()|%{$_.name;$sizenamehash.$($_.name).getenumerator()}|group value|?{$_.count -gt 1})|%{write-host "Duplicates:" -fore green;$_.group.name;$m+=$_.group.name.count}
 
 ""
 write-host "Number of Files checked: $($files.count)."
 write-host "Number of duplicate files: $m."
 ""
 write-host "Time to run: $(([datetime]::now)-$starttime|select hours, minutes, seconds, milliseconds)"
 ""
`

