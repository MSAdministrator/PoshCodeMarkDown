---
Author: angel-keeper
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1877
Published Date: 2010-05-26t23
Archived Date: 2013-05-27t19
---

# seach-localgroupmember - 

## Description

search users in the local administrators group on computers domain network

## Comments



## Usage



## TODO



## function

`seach-localgroupmemberdomennetwork`

## Code

`#
 #
 function Seach-LocalGroupMemberDomenNetwork() {
 param(
 $Domen,
 $User
 )
 
 function Ping ($Name){ 
     $ping = new-object System.Net.NetworkInformation.Ping
     if ($ping.send($Name).Status -eq "Success") {$True}
     else {$False} 
 	trap {Write-Verbose "Error Ping"; $False; continue}
 }
 
 [string[]]$Info
 [string[]]$Computers
 
 if ($User){$Connection = Get-Credential -Credential $User}
 
 $Computers = Get-QADComputer -Service $Domen -OSName '*XP*','*Vista*','*7*' -SizeLimit 0 -ErrorAction SilentlyContinue | 
 			 Select-Object name -ExpandProperty name
 
 Foreach ($Computer in $Computers){
 	$Alive = Ping $Computer
 	if ($Alive -eq "True"){
 		
 		if ($Connection) {
 			Trap {Write-Host "Error WMI $Computer";Continue}
 			$GroupName = Get-WmiObject win32_group -ComputerName $Computer -Credential $Connection | 
 						Where-Object {$_.SID -eq 'S-1-5-32-544'} | 
 						Select-Object name -ExpandProperty name
 		}
 		else {
 		Trap {Write-Host "Error WMI $Computer";Continue}
 			$GroupName = Get-WmiObject win32_group -ComputerName $Computer | 
 						Where-Object {$_.SID -eq 'S-1-5-32-544'} | 
 						Select-Object name -ExpandProperty name
 		}
 		
 		if ($GroupName){
 			$Users = ([ADSI]"WinNT://$Computer/$GroupName").psbase.invoke("Members") | 
 						% {$_.GetType().InvokeMember("Name", 'GetProperty', $null, $_, $null)}
 			$Info += $Users | % {$_ | Select-Object @{e={$Computer};n='Computer'},@{e={$_};n='Login'}}
 		}
 	}
 }
 $Info
 }
`

