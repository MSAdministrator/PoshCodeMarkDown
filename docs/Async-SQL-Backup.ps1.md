---
Author: justin
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2787
Published Date: 2012-07-13t12
Archived Date: 2017-05-30t19
---

# async sql backup - 

## Description

this will run multiple backup jobs at the same time. this script is cluster aware.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $backuppath = "\\server\sqlbackups\"
 $alertaddress = "jrich523@domain.com"
 $smtp = "smtp.domain.com"
 $retaindays = 14
 $hname = (gwmi win32_computersystem).name
 $errorstate = 0
 $body =@()
 $backups = @()
 $conns = @()
 $completed = @{}
 
 [System.Reflection.Assembly]::LoadWithPartialName("Microsoft.SqlServer.SMO") | Out-Null
 [System.Reflection.Assembly]::LoadWithPartialName("Microsoft.SqlServer.SmoExtended") | Out-Null
 $dt = get-date -format "MMddyy"
 
 $Instances = Get-Item "HKLM:\software\microsoft\microsoft sql server\instance names\sql"
 
 foreach($InstanceName in $Instances.property)
 {
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
 	$backuppath += $sql | ?{$_.instancename -ne "" -and $_.instancename -ne "MSSQLSERVER"} | %{$_.instancename + "\"}
 	$dbs = $sql.databases | ? {!$_.isSystemObject}
 	#$dbs |?{$_.recoverymodel -eq [microsoft.sqlserver.management.smo.recoverymodel]::simple} | %{$_.recoverymodel = [microsoft.sqlserver.management.smo.recoverymodel]::Full;$_.alter()}
 
 
 
 	foreach ($db in $dbs)
 	{
 	    $path = $backuppath + $db.name + "\"
 	    if(!(Test-Path $path)){mkdir $path | Out-Null}
 	    $conn = New-Object Microsoft.SqlServer.management.Smo.Server $InstConn
 		$conn.ConnectionContext.StatementTimeout = 0
 	    $bk = new-object microsoft.sqlserver.management.smo.backup
 	    $bk.BackupSetDescription = "fullbackup of $($db.name) on $(get-date)"
 	    $bk.BackupSetName = "full"
 	    $bk.database = $db.name
 	    $bk.Devices.AddDevice("$backuppath$($db.name)\$($db.name)-$dt.bak",'File')
 	    $backups += $bk
 	    $index = $backups.length -1
 	    Register-ObjectEvent -InputObject $backups[$index] -EventName "Complete" -SourceIdentifier $db.name  -MessageData "$($db.name)-$index" | Out-Null
 	    Register-ObjectEvent -InputObject $backups[$index] -EventName "Information" -SourceIdentifier "info-$($db.name)"  -MessageData "$($db.name)-$index" | out-null
 	    $conns += $conn
 	    $backups[$index].SqlBackupAsync($conns[$index])
 	}
 }
 
 $timer = New-Object timers.Timer
 $action = {
 $global:backups | ?{$_.asyncstatus.executionstatus -ne "InProgress"}|?{$global:completed.($_.database) -eq 0}|%{$global:completed.($_.database) = 1;$global:body += "timer caught: $($_.database)";"timer kicked off: $($_.database)"} 
 	New-Event -SourceIdentifier timer
 }
 Register-ObjectEvent -InputObject $timer -Action $action -SourceIdentifier timercheck -EventName elapsed
 $timer.start()
 
 
 while(($completed.values | measure -sum).sum -lt $backups.length){
 	wait-event | Tee-Object -variable theevent | Remove-Event
 	
     if($theevent.sourceIdentifier -ne "timer")
 	{
 	$msg = $theevent.sourceeventargs.error.message
     $db,$index = $theevent.messagedata.split('-')
     $status = $theevent.sourceargs[0].asyncstatus.executionstatus    
 
 	if($completed.$db -ne 1)
     {
 
     	switch ($status){
     		"Succeeded" {
     			$completed.$db = 1
                 $body += "$db  successfully"
     			break;}
     		"Failed" {
     			$body += "$db FAILED: $msg"
     			$completed.$db, $errorstate = 1
     			break;}
     		"InProgress"{
     			break;}
     		"Inactive" {
     			break;}
     		}
         }
 	}
 }
 
 if($errorstate -eq 1){$subject = "DBBK: Failure on $hname"} else {$subject = "DBBK: Success on $hname"}
 Send-MailMessage -Subject $subject -BodyAsHtml ([string]::join("<br>",$body)) -From $alertaddress -To $alertaddress -SmtpServer $smtp
 
 $backups | %{$_.devices[0].name} | Split-Path | gci -Recurse | ? {!$_.PSIsContainer -and $_.lastWriteTime -lt [dateTime]::today.addDays(-1 * $retaindays)} | ri -force
`

