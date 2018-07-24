---
Author: james gentile
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2877
Published Date: 2012-07-28t17
Archived Date: 2012-01-12t06
---

# writefilename - 

## Description

write multiline overwriting messages, typically for iterating through long file names.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 
     if ($Host.Name -match 'ise') 
     { write-host $writestr; return }
                                                                             
     if ($global:wfnlastlen -eq $null) {$global:wfnlastlen=0}              
     $ctop=[console]::cursortop
     $cleft=[console]::cursorleft	
 
 	$oldwritestrlen=$writestr.length
     
     $rem=$null
 	$writelines = [math]::divrem($writestr.length+$cleft, [console]::bufferwidth, [ref]$rem)
 
     if ($cwe -gt 0) {$ctop-=($cwe)}
     
 	write-host "$writestr" -nonewline    
 	$global:wfnoldctop=[console]::cursortop
 	$global:wfnoldcleft=[console]::cursorleft
 	
 	if ($global:wfnlastlen -gt $writestr.length)
 	{
 	}
 	
 	
 	$global:wfnlastlen = $oldwritestrlen
 
     if ($ctop -lt 0) {$ctop=$cleft=0}
 	[console]::cursortop=$ctop
 	[console]::cursorleft=$cleft
 }
     if ($Host.Name -match 'ise') 
     { return }
     if ($global:wfnoldctop -ne $null -and $global:wfnoldcleft -ne $null) 
     {
         [console]::cursortop=$global:wfnoldctop
         [console]::cursorleft=$wfnoldcleft  
         if ($global:wfnoldcleft -ne 0 -and $switch)
         {
             write-host ""
         }
     }    
     $global:wfnoldctop=$null
     $global:wfnlastlen=$null
     $global:wfnoldcleft=$null
 }
 
     write-host "Checking: " -nonewline
     dir $args -recurse -ea 0 -force | %{WriteFileName ("$($_.fullname) ..."*(get-random -min 1 -max 100))}
     
     WriteFileNameEnd
     write-host "Done! exiting."
`

