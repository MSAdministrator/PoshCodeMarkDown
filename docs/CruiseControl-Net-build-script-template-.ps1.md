---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 3238
Published Date: 
Archived Date: 2015-04-23t01
---

#  - 

## Description

cruisecontrol.net build script template with detailed logging via the ccnetlistenerfile

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 Param(
 	[string]$CCNetListenerFile = $null,
 	[string]$MyComputerName = [System.Environment]::MachineName,
 	[string]$Config='Debug',
 	[int]$CCNetLogQueueLength = 10
 	[switch]$NoBuild,
 	[string]$SpecificVdir = $null
 )
 #...
 #<powershell>
 #<scriptDirectory>iphi</scriptDirectory>
 #<script>deploy.ps1</script>
 #<executable>C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe</executable>
 #<dynamicValues>
 #</dynamicValues>                    
 #</powershell>
 #...
 
 [Environment]::CurrentDirectory=(Get-Location -PSProvider FileSystem).ProviderPath
 
 function CCNetListenerLog( $q ) {
 	if($CCNetListenerFile -ne '' -and $CCNetListenerFile -ne $null) { 
 		$stream = [System.IO.StreamWriter] $CCNetListenerFile
 		$stream.AutoFlush = $true;
 		$stream.WriteLine("<data>")
 		foreach( $item in $q ) {
 			$stream.WriteLine($item)
 		}
 		$stream.WriteLine("</data>")
 		$stream.Close()
 	}
 }
 
 $q = New-Object System.Collections.Queue
 function LogBuildPoint( [string]$text ) {
 	if( $q.Count -ge $CCNetLogQueueLength ) {
 		$old = $q.Dequeue();
 	}
 	$stamp = [System.DateTime]::Now.ToString("yyyy-MM-dd hh:mm:ss");
 	$item = "<Item Time='$stamp' Data='$text' />"
 
 	$q.Enqueue( $item );
 	Write-Host $text 
 	if($CCNetListenerFile -ne '' -and $CCNetListenerFile -ne $null) { 
 		CCNetListenerLog $q 
 	}
 }
 
 function pd( [string]$location ) {
 	Push-Location $location;
 	[Environment]::CurrentDirectory=(Get-Location -PSProvider FileSystem).ProviderPath;
 }
 function ppd() {
 	Pop-Location
 	[Environment]::CurrentDirectory=(Get-Location -PSProvider FileSystem).ProviderPath;
 }
 
 function Build() {
 	$total = ($repetition_object | Measure-Object).Count
 	$num = 0
 	$startTime = [System.DateTime]::Now
 	$repetition_object | % {
 		$_ | RepetitiveTask
 		
 		$num++
 		$curTime = [System.DateTime]::Now
 		$percent = ($num / $total)
 		$expected = ($startTime + (New-Object TimeSpan (($curTime - $startTime).Ticks / $percent))).ToShortTimeString()
 		LogBuildPoint "Finished $num out of $total, expect to complete at: $expected"		
 	}
 }
 
 if(!$NoBuild) {
 	Build
 }
`

