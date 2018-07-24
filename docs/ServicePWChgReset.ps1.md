---
Author: steven saehrig
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 552
Published Date: 2009-08-27t02
Archived Date: 2014-02-12t08
---

# servicepwchgreset - 

## Description

requires – quest activeroles snapin

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #
 
 function func_Forest()
 {
 [System.DirectoryServices.ActiveDirectory.Forest]::GetCurrentForest().Domains | ForEach-Object {
 	Get-QADComputer -Service $_.Name -SizeLimit 0 -ErrorAction SilentlyContinue `
 		| Add-Member -Name 'DomainName' -MemberType NoteProperty -Value $_.Name -PassThru
 }
 }
 
 $Array = @()
 func_forest | where { $_.Name  -like  'jxr*' } | Sort-Object -property "Name" |? { $array += $_.name}
 $file = $array | Out-File -FilePath c:\txt\host.txt -Append
 
 
 $StartName = "username"
 $csvlocation = "c:\txt\service.csv"
 $txtlocation = "c:\txt\service.txt"
 $computer = gc c:\txt\host.txt
 $password = "password"
 $service = gwmi -Class Win32_Service -namespace root\CIMV2 -ComputerName $computer | Where-Object {$_.StartName -match $startname}                   
 
 $service | Export-Csv $csvlocation
 
 	foreach ($i in $service) {
 			($i | FT systemname, Displayname, State, Startname, Status | Out-File -Append -FilePath $txtlocation )
 	}
 }
 
 	foreach ($i in $service) {
             (Write-Host -ForegroundColor "Yellow" "Changing password on" $i.SystemName "Service Name"$i.Name) 
 			($i.Change($Null,$Null,$Null,$Null,$Null,$Null,$Null,$password))
 			(Write-Host -ForegroundColor "green" "Password Successfully Changed on" $i.SystemName "Service Name"$i.Name) 
 	}
 }		
 
 $s = gwmi -Class Win32_Service -namespace root\CIMV2 -ComputerName $computer | Where-Object {$_.StartName -match $startname}
  	foreach ($i in $s) {
 			if ($i.State -eq "running") {
 				Write-Host -ForegroundColor "Yellow" "Service name" $i.SystemName "Service name"$i.Name "is" $i.state 
 					$i.StopService()
 					$b = gwmi -Class Win32_Service -namespace root\CIMV2 -ComputerName $computer | Where-Object {$_.StartName -match $startname}
 						Write-Host -ForegroundColor "RED" "Service name" $b.SystemName "Service name" $b.Name "is" $b.state 
 					$i.StartService()
 				    $c = gwmi -Class Win32_Service -namespace root\CIMV2 -ComputerName $computer | Where-Object {$_.StartName -match $startname}
 					Write-Host -ForegroundColor "Green" "Server name" $c.SystemName  "Service name" $c.Name "is" $c.state }
 				
 			elseif ($i.State -eq "Stopped") {
 				Write-Host -ForegroundColor "RED" "Service name" $i.SystemName "Service name" $i.Name $i.state "Service will not be Started"  }
 	}
 }
 
 
 exporttxt
 changepw | out-null
 restartsvc | out-null
`

