---
Author: justin
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2788
Published Date: 2011-07-13t12
Archived Date: 2011-07-16t11
---

# sql log backup - 

## Description

this is the log backup script to be used with the async backup script http

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 $backuppath = "\\server\sqlbackups\"
 $alertaddress = "jrich@domain.com"
 $smtp = "smtp.domain.com"
 $hname = (gwmi win32_computersystem).name
 [System.Reflection.Assembly]::LoadWithPartialName("Microsoft.SqlServer.SMO") | Out-Null
 [System.Reflection.Assembly]::LoadWithPartialName("Microsoft.SqlServer.SmoExtended") | Out-Null
 $dt = get-date -format "MMddyy-hhmm"
 
 
 $Instances = Get-Item "HKLM:\software\microsoft\microsoft sql server\instance names\sql"
 
 foreach($InstanceName in $Instances.property)
 {
 	$body =@()
 	$errorstate = 0
 	$InstancePath = $Instances.GetValue($InstanceName)
 	if(Test-Path ("HKLM:\software\microsoft\microsoft sql server\" + $InstancePath + "\cluster"))
 		$ServerName = (gp ("HKLM:\software\microsoft\microsoft sql server\" + $InstancePath + "\cluster")).ClusterName
 	}
 	{
 		$ServerName = $hname
 	}
 	
 	if($InstanceName -eq "MSSQLSERVER")
 		$InstConn = $ServerName
 	}
 	else
 		$InstConn = $ServerName + "\" + $InstanceName
 	}
 
 
 	$sql = New-Object Microsoft.SqlServer.management.Smo.Server $InstConn
 	$backuppath += $sql.name + "\"
 	$backuppath += $sql | ?{$_.instancename -ne "" -or $_.instancename -ne "MSSQLSERVER"} | %{$_.instancename + "\"}
 	$dbs = $sql.databases | ? {!$_.isSystemObject -And $_.recoverymodel -eq  [microsoft.sqlserver.management.smo.recoverymodel]::full}
 	
 	
 	foreach ($db in $dbs)
 	{
 		$path = $backuppath + $db.name + "\"
 		$lwt = (gci $path *.bak |sort lastwritetime | select -Last 1).lastwritetime
 		gci $path $.trn | where {$_.lastwritetime -le $lwt} | ri -force
 		
 		if(!(Test-Path $path)){mkdir $path | Out-Null}
 	    $conn = New-Object Microsoft.SqlServer.management.Smo.Server $InstConn
 	    $bk = new-object microsoft.sqlserver.management.smo.backup
 	    $bk.BackupSetDescription = "log backup of $($db.name) at $(get-date)"
 		$bk.action = [microsoft.SqlServer.management.smo.backupactiontype]::log
 	    $bk.BackupSetName = "log backup"
 	    $bk.database = $db.name
 	    $bk.Devices.AddDevice("$backuppath$($db.name)\$($db.name)-$dt.trn",'File') 
 		try{
 		$msg = "$($bk.database) completed successfully"
 		$bk.sqlbackup($sql)}
 		catch{$errorstate = 1; $msg = "$($bk.database) Failed : $_"}
 		$body += $msg
 	}
 	
 	if($errorstate -eq 1){$subject = "Log Failure on $hname"} else {$subject = "Log Success on $hname"}
 	Send-MailMessage -Subject $subject -BodyAsHtml ([string]::join("<br>",$body)) -From $alertaddress -To $alertaddress -SmtpServer $smtp
 	
 }
`

