---
Author: stephen wheet
Publisher: 
Copyright: 
Email: 
Version: 4.0
Encoding: ascii
License: cc0
PoshCode ID: 4985
Published Date: 2014-03-13t20
Archived Date: 2014-03-21t14
---

# remove broken ntfs perm - 

## Description

comment

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #==========================================================================
 #
 #
 #
 #
 #
 #==========================================================================
 
 Function checkshare {
    Param($PassedShareName)
    Process 
    {
             
             $path = "\\$serverFQDN\$PassedShareName"
             $filename = $path.Replace("\","-") + ".csv"
 
 
             $list = "Task,Path,Access Entity,"
             $list | format-table | Out-File "c:\reports\unknownsids\$filename"
 
             Write-Host "Writing results to : $filename"
             $date = Get-Date
             Write-Host $date
             Write-Host "Getting Folders in:  $Path"
             [Array] $folders = Get-ChildItem -path $path -recurse | ? {$_.PSIsContainer} 
 
 
             ForEach ($folder in [Array] $folders)
             {
 				If ($folder.pspath){
 					$PSPath = (Convert-Path $folder.pspath)
                 	Write-Host "Checking Dir:  $PSPath"
     			
 					If ($PSPath){
 						$error.clear()
                 		$acl = Get-Acl -Path $PSPath
 						if (!$?) {
                 	    	$errmsg = "Error,$PSPath,ACCESS DENIED"
                 	    	$errmsg | format-table | Out-File -append "$filename"
     						
                 		$ACL.Access |
 						?{!$_.IsInherited} |
 						?{ $_.IdentityReference -like $unknownsid -or $_.IdentityReference -like $olddomain } |
 						% {$value = $_.IdentityReference.Value
                 	
 							$localsid = 0
 							If ($value -like $unknownsid){
                         		$checkforlocal = $value.split("-")
                        	 		$total =$checkforlocal.count -1
                         		if ($checkforlocal[$total] -match "100?" -or $checkforlocal[$total] -match "500"){
                          		   	$localsid = 1
 								   	#$list = ("Local,$PSPath,$value")
 									#$list | format-table | Out-File -append "$filename"
 							
                     		If (!$localsid){
                                     Write-host "Found - $PSPath - $value"
 									$list = ("Found,$PSPath,$value")
                           			$list | format-table | Out-File -append "$filename"
                             		
                             		if ($removeacl) { $ACL.RemoveAccessRuleSpecific($_)
                                         Write-host "Deleting - $PSPath - $value"
                             	    	$list = ("Deleting,$PSPath,$value")
                             	    	$list | format-table | Out-File -append "$filename"
                         
                             		if ($removeacl -and $value) {
                                         $date = Get-Date
                                         Write-Host $date
                                         Write-host "Setting - $PSPath"
                             	    	$list = ("Setting,$PSPath")
                              		   	$list | format-table | Out-File -append "$filename"
                                 		Set-ACL $PSpath $ACL
                                 		$value = ""
 
 get-pssnapin -registered | add-pssnapin -passthru
 $ReqVersion = [version]"1.2.2.1254" 
 
 if($QadVersion -lt $ReqVersion) { 
     throw "Quest AD cmdlets version '$ReqVersion' is required. Please download the latest version" 
 
 $value = ""
 $ErrorActionPreference = "SilentlyContinue"
 $localsid = 0
 
 
 
 
 Foreach ($server in $Servers ) {
 	$serverFQDN = $server.dnsname
 	write-host  $serverFQDN
 
 	$ping = new-object System.Net.NetworkInformation.Ping
 	$Reply = $ping.Send($serverFQDN)
     if ($Reply.status -eq "Success"){
         Write-Host "Online"
 
         If ($serverFQDN -like "*netapp*"){
             $server = new-object netapp.manage.naserver($serverFQDN,1,0)
 			
             $server.Style = "RPC"
 			
             $NaElement = New-Object NetApp.Manage.NaElement("system-cli")
             $arg = New-Object NetApp.Manage.NaElement("args")
             $arg.AddNewChild('arg','cifs')
             $arg.AddNewChild('arg','shares')
             $NaElement.AddChildElement($arg)
             $CifsString = $server.InvokeElem($NaElement).GetChildContent("cli-output")
 			
             $null,$null,$Lines = $CifsString.Split("`n")
 
             Foreach ($line in $Lines ) {
                                 
 				if (!$line.startswith("			") -and $line -notlike "*Etc$*" -and $line -notlike "*C$*"){
                     $l= $line.Split(" ")
 		
                
             Get-WMIObject Win32_Share -Computer $serverFQDN | 
             	where {$_.Name -like "*$*" -and $_.Name -notlike "*ADMIN*" -and $_.Name -notlike "*IPC*" -and $_.Name -notlike "*lib*"} |
 				%{
             		$sharename = $_.name
`

