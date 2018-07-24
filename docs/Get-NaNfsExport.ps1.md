---
Author: glnsize
Publisher: 
Copyright: 
Email: 
Version: 1.8
Encoding: ascii
License: cc0
PoshCode ID: 923
Published Date: 2009-03-08t22
Archived Date: 2015-11-14t04
---

# get-nanfsexport - 

## Description

function that will retrieve all the nfs exports from a netapp san.  example usage of thee ontap sdk 3.5 avaliable here http

## Comments



## Usage



## TODO



## function

`get-nanfsexport`

## Code

`#
 #
 #
 #
 #
 
 Function Get-NaNfsExport {
     Param(
         [NetApp.Manage.NaServer]
         $Server,
         [String]
         $Path
     )
     Begin {
         $out = @()
     }
     Process {
         trap [NetApp.Manage.NaAuthException] {
             Write-Error "Bad login/password".
             break
         }
         $NaOut = New-Object NetApp.Manage.NaElement("nfs-exportfs-list-rules")
               
         $NaResults = $Server.InvokeElem($NaOut).GetChildByName("rules").getchildren()    
         
         foreach ($NaElement in $NaResults) {
             $NaNFS = "" | Select-Object Path, ActualPath, ReadOnly, ReadWrite, Root, Security
             $NaNFS.Path = $NaElement.GetChildContent("pathname")
             
             switch ($NaElement) {
                 {$_.GetChildByName("read-only")}
                 	{	
                 		$ReadOnly = ($_.GetChildByName("read-only")).getchildren()
                 		foreach ($read in $ReadOnly) {
                 			If ($read.GetChildContent("all-hosts")) {
                 				$roList = 'All-Hosts'
                 			}
                 			Elseif ($read.GetChildContent("name")) {
                 				$roList += $read.GetChildContent("name")
                 			}
                 		}
                 		$NaNFS.ReadOnly = $roList
                 	}
                 {$_.GetChildByName("read-write")}
                 	{	
                         $ReadWrite = ($_.GetChildByName("read-write")).getchildren()
                 		foreach ($write in $ReadWrite) {
                 			If ($write.GetChildContent("all-hosts")) {
                 				$rwList = 'All-Hosts'
                 			}
                 			Elseif ($r.GetChildContent("name")) {
                 				$rwList += $write.GetChildContent("name")
                 			}
                 		}
                 		$NaNFS.ReadWrite = $rwList
                 	}
                 {$_.GetChildByName("root")}
                 	{
                 		$Root = ($_.GetChildByName("root")).getchildren()
                 		foreach ($r in $Root) {
                 			If ($r.GetChildContent("all-hosts")) {
                 				$rrList = 'All-Hosts'
                 			}
                 			Elseif ($r.GetChildContent("name")) {
                 				$rrList += $r.GetChildContent("name")
                 			}
                 		}
                 		$NaNFS.Root = $rrList
                 	}
                 {$_.GetChildByName("sec-flavor")}
                 	{
                 		$Security = ($_.GetChildByName("sec-flavor")).getchildren()
                 		foreach ($s in $Security) {
                 			if ($r.GetChildContent("flavor")) {
                 				$SecList += $r.GetChildContent("flavor")
                 			}
                 		}
                 		$NaNFS.Security = $SecList
                 	}
                 {$_.GetChildByName("actual-pathname")}
                     {
                     
                     	$NaNFS.ActualPath = $_.GetChildByName("actual-pathname")
                     }
                 {!$_.GetChildByName("actual-pathname")}
                     {
                     
                     	$NaNFS.ActualPath = $_.GetChildContent("pathname")
                     }
             }
             $out += $NaNFS
         }
     }
     End {
         If ($Path) {
             return $out | ?{$_.Path -match $Patch}
         } 
         else {
             return $out
         }
     }
 }
`

