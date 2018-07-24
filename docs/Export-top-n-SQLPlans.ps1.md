---
Author: alzdba
Publisher: 
Copyright: 
Email: 
Version: 1.00
Encoding: ascii
License: cc0
PoshCode ID: 3732
Published Date: 2013-10-31t07
Archived Date: 2013-05-09t04
---

# export top n sqlplans - 

## Description

export top n consuming sqlplans via avg_worker_time (=cpu) for all databases of a given sqlserver (sql2005+) instance

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 ALZDBA SQLServer_Export_SQLPlans_SMO.ps1
 Export top n consuming sqlplans via avg_worker_time (=cpu) for all databases of a given SQLServer (SQL2005+) Instance
 results in a number of .SQLPlan files and the consumption overview .CSV file
 #>
 $SQLInstance = 'uabe0db97\uabe0db97'
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
   
 Clear-Host
 
 
 [string[]]$ExcludedDb = 'tempdb' , 'model'
 
 $AllDbSQLPlan = $null 
 
 $SampleTime = Get-Date -Format 'yyyyMMdd_HHmm'
 [System.Reflection.Assembly]::LoadWithPartialName("Microsoft.SqlServer.SMO") | Out-Null 
 
 $serverInstance = New-Object ("Microsoft.SqlServer.Management.Smo.Server") $SQLInstance
 $serverInstance.ConnectionContext.ApplicationName = "DBA_Export_SQLPlans"
 $serverInstance.ConnectionContext.WorkstationId = $env:COMPUTERNAME 
 $serverInstance.ConnectionContext.ConnectTimeout = 5
 $serverInstance.ConnectionContext.Connect()
 $Query = ''
 if ( $serverInstance.VersionMajor -lt 9 ) {
 	write-host $('{0} is of a Non-supported SQLServer version [{1}].' -f $SQLInstance, $serverInstance.VersionString) -ForegroundColor Black -BackgroundColor Red 
 	break
 
 	}
 elseif ( $serverInstance.VersionMajor -lt 10 ) {
 	$Query = $('WITH XMLNAMESPACES ( ''http://schemas.microsoft.com/sqlserver/2004/07/showplan'' AS PLN )
 			SELECT TOP ( {0} ) db_name() as Db_Name
 		  , Object_schema_name(qp.objectid ) as [Schema]
 	      , Object_name(qp.objectid ) AS [Object_Name]
 	      , ISNULL(qs.total_elapsed_time / qs.execution_count, 0) AS [Avg_Elapsed_Time]
 	      , qs.execution_count AS [Execution_Count]
 	      , qs.total_worker_time AS [Total_Worker_Time]
 	      , qs.total_worker_time / qs.execution_count AS [Avg_Worker_Time]
 	      , ISNULL(qs.execution_count / DATEDIFF(SS, qs.creation_time, GETDATE()), 0) AS [Calls_per_Second]
 	      , qs.max_logical_reads
 	      , qs.max_logical_writes
 		  , qs.creation_time as cached_time
 		  , qp.query_plan.exist(''/PLN:ShowPlanXML//PLN:MissingIndex'') as Missing_Indexes
 	      , DATEDIFF( SS, qs.creation_time, getdate()) as Time_In_Cache_SS
 		  , row_number() over ( order by qs.total_elapsed_time / qs.execution_count DESC ) [Row_Number]
 	      , qp.query_plan
 	FROM    sys.dm_exec_query_stats AS qs WITH ( NOLOCK )
 	CROSS APPLY sys.dm_exec_query_plan(qs.plan_handle) AS qp
 	WHERE   qp.dbid = DB_ID()
 	        and qs.execution_count > 5
 			and Object_name(qp.objectid ) not like  ''spc_DBA%''
 			and qs.total_worker_time / qs.execution_count > 50  /* in microseconds */
 	ORDER BY Avg_Elapsed_Time DESC
 	OPTION  ( RECOMPILE ) ; ' -f $nTop )
 	}
 elseif ( $serverInstance.VersionMajor -eq 10 -and $serverInstance.VersionMinor -lt 50 ) {
 	$Query = $('WITH XMLNAMESPACES ( ''http://schemas.microsoft.com/sqlserver/2004/07/showplan'' AS PLN )
 			SELECT TOP ( {0} ) db_name() as Db_Name
 		  , Object_schema_name(qp.objectid ) as [Schema]
 	      , Object_name(qp.objectid ) AS [Object_Name]
 	      , ISNULL(qs.total_elapsed_time / qs.execution_count, 0) AS [Avg_Elapsed_Time]
 	      , qs.execution_count AS [Execution_Count]
 	      , qs.total_worker_time AS [Total_Worker_Time]
 	      , qs.total_worker_time / qs.execution_count AS [Avg_Worker_Time]
 	      , ISNULL(qs.execution_count / DATEDIFF(SS, qs.creation_time, GETDATE()), 0) AS [Calls_per_Second]
 	      , qs.max_logical_reads
 	      , qs.max_logical_writes
 		  , qs.creation_time as cached_time
 		  , qp.query_plan.exist(''/PLN:ShowPlanXML//PLN:MissingIndex'') as Missing_Indexes
 	      , DATEDIFF( SS, qs.cached_time, getdate()) as Time_In_Cache_SS
 		  , row_number() over ( order by qs.total_elapsed_time / qs.execution_count DESC ) [Row_Number]
 	      , qp.query_plan
 	FROM    sys.dm_exec_query_stats AS qs WITH ( NOLOCK )
 	CROSS APPLY sys.dm_exec_query_plan(qs.plan_handle) AS qp
 	WHERE   qp.dbid = DB_ID()
 	        and qs.execution_count > 5
 			and Object_name(qp.objectid ) not like  ''spc_DBA%''
 	and qs.total_worker_time / qs.execution_count > 50000 /* in microseconds */
 	ORDER BY Avg_Elapsed_Time DESC
 	OPTION  ( RECOMPILE ) ; ' -f $nTop )
 	}
 else {
 	$Query = $('WITH XMLNAMESPACES ( ''http://schemas.microsoft.com/sqlserver/2004/07/showplan'' AS PLN )
 			SELECT TOP ( {0} ) db_name() as Db_Name
 		  , Object_schema_name(p.object_id ) as [Schema]
 	      , p.name AS [Object_Name]
 	      , qs.total_elapsed_time / qs.execution_count AS [avg_elapsed_time]
 	      , qs.total_elapsed_time
 	      , qs.execution_count
 	      , cast(ISNULL(qs.execution_count * 1.00 / DATEDIFF(SS, qs.cached_time, GETDATE()), 0) as decimal(9,3)) AS [CallsPerSecond]
 	      , qs.total_worker_time / qs.execution_count AS [Avg_Worker_Time]
 	      , qs.total_worker_time AS [Total_Worker_Time]
 		  , qp.query_plan.exist(''/PLN:ShowPlanXML//PLN:MissingIndex'') as Missing_Indexes
 	      , qs.cached_time
 	      , DATEDIFF( SS, qs.cached_time, getdate()) as Time_In_Cache_SS
 		  , row_number() over ( order by qs.total_elapsed_time / qs.execution_count DESC ) [Row_Number]
 	      , qp.query_plan
 	FROM    sys.procedures AS p WITH ( NOLOCK )
 	INNER JOIN sys.dm_exec_procedure_stats AS qs WITH ( NOLOCK )
 	        ON p.[object_id] = qs.[object_id]
 	CROSS APPLY sys.dm_exec_query_plan(qs.plan_handle) AS qp
 	WHERE   qs.database_id = DB_ID()
 	        and qs.execution_count > 5
 			/* only non-ms sprocs */
 	        and p.is_ms_shipped = 0
 			and p.name not like  ''spc_DBA%''
 			and qs.total_worker_time / qs.execution_count > 50000  /* in microseconds */
 
 	ORDER BY avg_elapsed_time DESC
 	OPTION  ( RECOMPILE ) ;' -f $nTop )
 	}
 
 $i = 0
 foreach($db in $serverInstance.Databases){
 	if ( $ExcludedDb -notcontains $db.Name ) {
 		$i += 1
 		$pct = 100 * $i / $serverInstance.Databases.Count 
 		Write-progress -Status "Processing DBs - $($db.Name)"  -Activity "Collection SQLPlans $SQLInstance" -PercentComplete $pct
 		try {
 			$DbSQLPlans = $db.ExecuteWithResults("$Query").Tables[0] 
 		
 			if ( !( $AllDbSQLPlan )) {
 				$AllDbSQLPlan = $DbSQLPlans.clone()
 				}
 				
 			if ( $DbSQLPlans.rows.count -gt 0 ) {
 				$AllDbSQLPlan += $DbSQLPlans
 				}
 			}
 		catch {
 			Write-Verbose $_.Exception.Message 
 			}
 		}
 	else {
 		Write-Verbose "Excluded Db $db.name"
 		}
 	}
 $serverInstance.ConnectionContext.Disconnect()
 
 	
 if ( $AllDbSQLPlan -and $AllDbSQLPlan.Count -gt 0 ) {
 	$TargetPath = "c:\tempo\Powershell"
 	if ( !(Test-Path $TargetPath) ) {
 		md $TargetPath | Out-Null 
 		}
 	Write-progress -Status "Exporting Consumption Data"  -Activity "Exporting SQLPlans $SQLInstance" -PercentComplete 15
 
 	$TargetFile = $('{0}-{1}_AllDbSQLPlan.csv' -f $SampleTime, $($SQLInstance -replace '\\', '_') )
 	$AllDbSQLPlan | Select Db_Name, Row_Number, Schema, Object_Name, avg_elapsed_time, total_elapsed_time, execution_count, Calls_Per_Second, Avg_Worker_Time, Total_Worker_Time, Missing_Indexes, cached_time, Time_In_Cache_SS | sort Db_Name, Row_Number | Export-Csv -Path $( Join-Path -Path $TargetPath -ChildPath $TargetFile ) -Delimiter ';' -NoTypeInformation 
 
 	$i = 0
 	foreach ( $p in $AllDbSQLPlan ) {
 		$i += 1
 		$pct = 100 * $i / $AllDbSQLPlan.Count 
 		Write-progress -Status "Exporting SQLPlan - $($p.Object_Name)"  -Activity "Exporting SQLPlans $SQLInstance" -PercentComplete $pct
 
 		$TargetFileName = $('{0}-{1}-{2}-{3}-{4}-{5}.SQLPlan' -f $SampleTime, $($SQLInstance.Replace('\','_')) , $p.Db_Name, $p.Row_Number, $p.Schema , $p.Object_Name )
 		Write-verbose $TargetFileName
 		
 		Out-File -FilePath $( Join-Path -Path $TargetPath -ChildPath $TargetFileName ) -InputObject $p.query_plan		
 		}
 
 	Invoke-Item  "$TargetPath"
 	}
 else {
 	Write-Host "No SQLPlans to be exported ! " -ForegroundColor Black -BackgroundColor Yellow
 	}
`

