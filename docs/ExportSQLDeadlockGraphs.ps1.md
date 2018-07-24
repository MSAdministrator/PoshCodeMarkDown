---
Author: alzdba
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4124
Published Date: 2013-04-21t08
Archived Date: 2013-04-24t00
---

# exportsqldeadlockgraphs - 

## Description

export top n deadlock graphs for all databases of a given sqlserver (sql2008+) instance

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 ALZDBA SQLServer_Export_DeadlockGraphs_SMO.ps1
 Export top n Deadlock graphs for all databases of a given SQLServer (SQL2008+) Instance
 results in a number of .XDL files and one overview file *AllDbDeadlockGraphs.csv.
 
 Double clicking an .XDL file will open the graphical representation of the deadlock info in SQLServer Management Studio
 ( You'll need SSMS 2012 to overcome the XML-schema changes they did during the SQL2008 lifetime )
 
 #>
 $SQLInstance = 'Yourserver\yourinstance' 
 [int]$nTop = 50
 
 trap {
   $err = $_.Exception
   write-verbose "Trapped error: $err.Message" 
   while( $err.InnerException ) {
 	   $err = $err.InnerException
 	   write-host $err.Message -ForegroundColor Black -BackgroundColor Red 
 	   };
   break
   }
 
 $ALLDbResults = $null 
 
 Clear-Host
 
 if (!($ALLDbResults) ) {
 [string[]]$ExcludedDb = 'master', 'model', 'msdb', 'DDBAserverPing'
 
 #$ALLDbResults = $null 
 
 $SampleTime = Get-Date -Format 'yyyyMMdd_HHmm'
 [System.Reflection.Assembly]::LoadWithPartialName("Microsoft.SqlServer.SMO") | Out-Null 
 
 $serverInstance = New-Object ("Microsoft.SqlServer.Management.Smo.Server") $SQLInstance
 $serverInstance.ConnectionContext.ApplicationName = "DBA_Export_DeadlockGraphs"
 $serverInstance.ConnectionContext.WorkstationId = $env:COMPUTERNAME 
 $serverInstance.ConnectionContext.ConnectTimeout = 5
 $serverInstance.ConnectionContext.Connect()
 
 $Query = ''
 if ( $serverInstance.VersionMajor -lt 10 ) {
 	write-host $('{0} is of a Non-supported SQLServer version [{1}].' -f $SQLInstance, $serverInstance.VersionString) -ForegroundColor Black -BackgroundColor Red 
 	break
 
 	}
 elseif ( $serverInstance.VersionMajor -eq 10 -and $serverInstance.VersionMinor -lt 50 ) {
 	write-host $('{0} is of a Non-supported SQLServer version [{1}].' -f $SQLInstance, $serverInstance.VersionString) -ForegroundColor Black -BackgroundColor Red 
 	break
 
 	}
 else {
 $Query = $( 'Set QUOTED_IDENTIFIER ON;
 /* Fetch the Health Session data into a temporary table */
 begin
 	 ( PK int not null primary key clustered,
 	  XMLDATA xml not null
 	  )
 
 	SELECT cast( 1 as int ) as PK, CAST(xet.target_data AS XML) AS XMLDATA
 	FROM    sys.dm_xe_session_targets xet
 	INNER JOIN    sys.dm_xe_sessions xe
 			ON ( xe.address = xet.event_session_address )
 	WHERE   xe.name = ''system_health'';
 
 	CREATE PRIMARY XML INDEX PXML_SystemHealthSessionData
 end ;
 ;
 with    cteDeadLocks
   as (
        SELECT  top ( 50 )
                 x.y.value(''(@timestamp)[1]'', ''datetime'') [timestampUTC]
               , CAST(x.y.value(''data(.//data/value)[1]'', ''VARCHAR(MAX)'') AS XML).value(''deadlock[1]/victim-list[1]/victimProcess[1]/@id[1]''
 , ''VARCHAR(128)'') [victim]
               , CAST(x.y.value(''data(.//data/value)[1]'', ''VARCHAR(MAX)'') AS XML).query(''deadlock/process-list/*'') [ProcessList]
               , CAST(x.y.value(''data(.//data/value)[1]'', ''VARCHAR(MAX)'') AS XML).query(''deadlock/resource-list/*'') [ResourceList]
               , CAST(x.y.value(''data(.)[1]'', ''VARCHAR(MAX)'') AS XML).query(''*'') as DeadlockGraph
        CROSS APPLY [XMLDATA].nodes(''/RingBufferTarget/event'') AS x ( y )
 WHERE    x.y.query(''.'').exist(''/event[@name="xml_deadlock_report"]'') = 1
 and  x.y.value(''(@timestamp)[1]'', ''datetime'') > DATEADD(dd, datediff(dd, 0, getUTCdate()) - 2 , 0)
 and  CAST(x.y.value(''data(.//data/value)[1]'', ''VARCHAR(MAX)'') AS XML).query(''deadlock/resource-list/*'').value(''(//@dbid)[1]'', ''integer'') = 
 db_id()
        Order by [timestampUTC] desc     
        )
     Select convert(varchar(25),SERVERPROPERTY(''ServerName'')) as ServerName
 		, *
 		/* process block 1 */
 		, [ResourceList].value(''(//@dbid)[1]'', ''integer'') [DbId1]
         , DB_NAME( [ResourceList].value(''(//@dbid)[1]'', ''integer'')) [DbName1]
         , [ProcessList].value(''(//@id)[1]'', ''nvarchar(128)'') [Id1]
         , [ProcessList].value(''(//@status)[1]'', ''nvarchar(128)'') [Status1]
         , [ProcessList].value(''(//@waitresource)[1]'', ''nvarchar(128)'') [waitresource1]
         , [ProcessList].value(''(//@waittime)[1]'', ''integer'') [waittime1]
         , [ProcessList].value(''(//@transactionname)[1]'', ''nvarchar(128)'') [transactionname1]
         , [ProcessList].value(''(//@clientapp)[1]'', ''nvarchar(256)'') [ApplicationName1]
         , [ProcessList].value(''(//@hostname)[1]'', ''nvarchar(256)'') [hostname1]
         , [ProcessList].value(''(//@loginname)[1]'', ''nvarchar(256)'') [loginname1]
         , [ProcessList].value(''(//@isolationlevel)[1]'', ''nvarchar(256)'') [isolationlevel1]
         , [ProcessList].value(''(//@currentdb)[1]'', ''integer'') [ConnectionCurrentDb1]
         , DB_NAME( [ProcessList].value(''(//@currentdb)[1]'', ''integer'')) [ConnectionCurrentDbName1]
         , [ProcessList].value(''(//@lasttranstarted)[1]'', ''datetime'') [lasttranstarted1]
         , [ProcessList].value(''(//@lastbatchstarted)[1]'', ''datetime'') [lastbatchstarted1]
 
  		/* process block 2 */
 		, [ResourceList].value(''(//@dbid)[2]'', ''integer'') [DbId2]
         , DB_NAME( [ResourceList].value(''(//@dbid)[2]'', ''integer'')) [DbName2]
         , [ProcessList].value(''(//@id)[2]'', ''nvarchar(128)'') [Id2]
         , [ProcessList].value(''(//@status)[2]'', ''nvarchar(128)'') [Status2]
         , [ProcessList].value(''(//@waitresource)[2]'', ''nvarchar(128)'') [waitresource2]
         , [ProcessList].value(''(//@waittime)[2]'', ''integer'') [waittime2]
         , [ProcessList].value(''(//@transactionname)[2]'', ''nvarchar(128)'') [transactionname2]
         , [ProcessList].value(''(//@clientapp)[2]'', ''nvarchar(256)'') [ApplicationName2]
         , [ProcessList].value(''(//@hostname)[2]'', ''nvarchar(256)'') [hostname2]
         , [ProcessList].value(''(//@loginname)[2]'', ''nvarchar(256)'') [loginname2]
         , [ProcessList].value(''(//@isolationlevel)[2]'', ''nvarchar(256)'') [isolationlevel2]
         , [ProcessList].value(''(//@currentdb)[2]'', ''integer'') [ConnectionCurrentDb2]
         , DB_NAME( [ProcessList].value(''(//@currentdb)[2]'', ''integer'')) [ConnectionCurrentDbName2]
         , [ProcessList].value(''(//@lasttranstarted)[2]'', ''datetime'') [lasttranstarted2]
         , [ProcessList].value(''(//@lastbatchstarted)[2]'', ''datetime'') [lastbatchstarted2]
 
  		/* process block 3 */
 		, [ResourceList].value(''(//@dbid)[3]'', ''integer'') [DbId3]
         , DB_NAME( [ResourceList].value(''(//@dbid)[3]'', ''integer'')) [DbName3]
         , [ProcessList].value(''(//@id)[3]'', ''nvarchar(128)'') [Id3]
         , [ProcessList].value(''(//@status)[3]'', ''nvarchar(128)'') [Status3]
         , [ProcessList].value(''(//@waitresource)[3]'', ''nvarchar(128)'') [waitresource3]
         , [ProcessList].value(''(//@waittime)[3]'', ''integer'') [waittime3]
         , [ProcessList].value(''(//@transactionname)[3]'', ''nvarchar(128)'') [transactionname3]
         , [ProcessList].value(''(//@clientapp)[3]'', ''nvarchar(256)'') [ApplicationName3]
         , [ProcessList].value(''(//@hostname)[3]'', ''nvarchar(256)'') [hostname3]
         , [ProcessList].value(''(//@loginname)[3]'', ''nvarchar(256)'') [loginname3]
         , [ProcessList].value(''(//@isolationlevel)[3]'', ''nvarchar(256)'') [isolationlevel3]
         , [ProcessList].value(''(//@currentdb)[3]'', ''integer'') [ConnectionCurrentDb3]
         , DB_NAME( [ProcessList].value(''(//@currentdb)[3]'', ''integer'')) [ConnectionCurrentDbName3]
         , [ProcessList].value(''(//@lasttranstarted)[3]'', ''datetime'') [lasttranstarted3]
         , [ProcessList].value(''(//@lastbatchstarted)[3]'', ''datetime'') [lastbatchstarted3]
  
   		/* process block 4 */
 		, [ResourceList].value(''(//@dbid)[4]'', ''integer'') [DbId4]
         , DB_NAME( [ResourceList].value(''(//@dbid)[4]'', ''integer'')) [DbName4]
         , [ProcessList].value(''(//@id)[4]'', ''nvarchar(128)'') [Id4]
         , [ProcessList].value(''(//@status)[4]'', ''nvarchar(128)'') [Status4]
         , [ProcessList].value(''(//@waitresource)[4]'', ''nvarchar(128)'') [waitresource4]
         , [ProcessList].value(''(//@waittime)[4]'', ''integer'') [waittime4]
         , [ProcessList].value(''(//@transactionname)[4]'', ''nvarchar(128)'') [transactionname4]
         , [ProcessList].value(''(//@clientapp)[4]'', ''nvarchar(256)'') [ApplicationName4]
         , [ProcessList].value(''(//@hostname)[4]'', ''nvarchar(256)'') [hostname4]
         , [ProcessList].value(''(//@loginname)[4]'', ''nvarchar(256)'') [loginname4]
         , [ProcessList].value(''(//@isolationlevel)[4]'', ''nvarchar(256)'') [isolationlevel4]
         , [ProcessList].value(''(//@currentdb)[4]'', ''integer'') [ConnectionCurrentDb4]
         , DB_NAME( [ProcessList].value(''(//@currentdb)[4]'', ''integer'')) [ConnectionCurrentDbName4]
         , [ProcessList].value(''(//@lasttranstarted)[4]'', ''datetime'') [lasttranstarted4]
         , [ProcessList].value(''(//@lastbatchstarted)[4]'', ''datetime'') [lastbatchstarted4]
     from    cteDeadLocks DL
     order by [timestampUTC] desc ; ' -f $nTop )
 	}
 	
 
 $i = 0
 foreach($db in $serverInstance.Databases){
 	if ( $ExcludedDb -notcontains $db.Name ) {
 		$i += 1
 		$pct = 100 * $i / $serverInstance.Databases.Count 
 		Write-progress -Status "Processing DBs - $($db.Name)"  -Activity "Collection DeadlockGraphs $SQLInstance" -PercentComplete $pct
 		try {
 			
 			$DbResults = $db.ExecuteWithResults("$Query").Tables[0] ;
 		
 			if ( !( $ALLDbResults )) {
 				$ALLDbResults = $DbResults.clone()
 				}
 				
 			if ( $DbResults.rows.count -gt 0 ) {
 				$ALLDbResults += $DbResults
 				}
 			
 			}
 		catch {
 			Write-host $_.Exception.Message -BackgroundColor Red -ForegroundColor Black 
 			}
 		}
 	else {
 		Write-Verbose "Excluded Db $db.name"
 		}
 	}
 $serverInstance.ConnectionContext.Disconnect()
 
 }
 	
 if ( $ALLDbResults -and $ALLDbResults.Count -gt 0 ) {
 	$TargetPath = "$env:TEMP\Powershell"
 	if ( !(Test-Path $TargetPath) ) {
 		md $TargetPath | Out-Null 
 		}
 	Write-progress -Status "Exporting Consumption Data"  -Activity "Exporting Deadlock Graphs $SQLInstance" -PercentComplete 15
 
 	$TargetFile = $('{0}-{1}_AllDbDeadlockGraphs.csv' -f $SampleTime, $($SQLInstance -replace '\\', '_') )
 	#$ALLDbResults | Select Db_Name, Row_Number, Schema, Object_Name, avg_elapsed_time, total_elapsed_time, execution_count, Calls_Per_Second, Avg_Worker_Time, Total_Worker_Time, Missing_Indexes, cached_time, Time_In_Cache_SS | sort Db_Name, Row_Number | Export-Csv -Path $( Join-Path -Path $TargetPath -ChildPath $TargetFile ) -Delimiter ';' -NoTypeInformation 
 	$ALLDbResults | SELECT ServerName, timestampUTC, victim, processlist, resourcelist, DbId1, DbName1, Id1, Status1, waitresource1, waittime1, transactionname1, ApplicationName1, hostname1, loginname1, isolationlevel1, ConnectionCurrentDb1, ConnectionCurrentDbName1, lasttranstarted1, lastbatchstarted1, DbId2, DbName2, Id2, Status2, waitresource2, waittime2, transactionname2, ApplicationName2, hostname2, loginname2, isolationlevel2, ConnectionCurrentDb2, ConnectionCurrentDbName2, lasttranstarted2, lastbatchstarted2, DbId3, DbName3, Id3, Status3, waitresource3, waittime3, transactionname3, ApplicationName3, hostname3, loginname3, isolationlevel3, ConnectionCurrentDb3, ConnectionCurrentDbName3, lasttranstarted3, lastbatchstarted3, DbId4, DbName4, Id4, Status4, waitresource4, waittime4, transactionname4, ApplicationName4, hostname4, loginname4, isolationlevel4, ConnectionCurrentDb4, ConnectionCurrentDbName4, lasttranstarted4, lastbatchstarted4 | sort TimestampUTC -desc | Export-Csv -Path $( Join-Path -Path $TargetPath -ChildPath $TargetFile ) -Delimiter ';' -NoTypeInformation 
 
 	$i = 0
 	foreach ( $p in $ALLDbResults ) {
 		$i += 1
 		$pct = 100 * $i / $ALLDbResults.Count 
 		Write-progress -Status "Exporting DeadlockGraphs - $($p.timestampUTC)"  -Activity "Exporting Deadlock Graphs $SQLInstance" -PercentComplete $pct
 		[datetime]$TimestampUTC = $p.timestampUTC
 
 		$TimestampUTCfmt = Get-Date -Date $TimestampUTC -format 'yyyyMMdd_hhmmss_fff'
 
 		$TargetFileName = $('DeadLockGraph-{0}-{1}-{2}.XDL' -f $($p.ServerName.Replace('\','_')), $p.DbName1, $TimestampUTCfmt )
 		Write-verbose $TargetFileName
 		
 		Out-File -FilePath $( Join-Path -Path $TargetPath -ChildPath $TargetFileName ) -InputObject $p.DeadlockGraph		
 		}
 
 	Invoke-Item  "$TargetPath"
 	}
 else {
 	Write-Host "No DeadlockGraphs to be exported ! " -ForegroundColor Black -BackgroundColor Yellow
 	}
 
 Write-Host 'The end.'
`

