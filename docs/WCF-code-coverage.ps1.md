---
Author: sixeyed
Publisher: 
Copyright: 
Email: 
Version: 10.0
Encoding: ascii
License: cc0
PoshCode ID: 3141
Published Date: 2012-01-04t10
Archived Date: 2012-02-01t10
---

# wcf code coverage - 

## Description

script for running unit tests over wcf services to get code coverage for the whole service stack

## Comments



## Usage



## TODO



## script

`get-apppool`

## Code

`#
 #
 param($msBuildTarget, $configurationName, [bool]$deleteInstrumentedAssemblies) 
 
 #-------------------------------------
 #-------------------------------------
 
 #-------------------------------------
 #-------------------------------------
 function get-apppool{
     [regex]$pattern="-ap ""($appPoolName)"""
     gwmi win32_process -filter 'name="w3wp.exe"' | % {
         $name=$_.name
         $cmd = $pattern.Match($_.commandline).Groups[1].Value
         $procid = $_.ProcessId
         New-Object psobject | Add-Member -MemberType noteproperty -PassThru Name $name |
             Add-Member -MemberType noteproperty -PassThru AppPoolID $cmd |
             Add-Member -MemberType noteproperty -PassThru PID $procid 
     }
 }
 
 #---------------------------------------
 #---------------------------------------
 function get-apppoolpid{
 	$appPoolPid = 0
 	$appPools = get-apppool
 	foreach ($appPool in $appPools){
 		if ($appPool.AppPoolID -eq "$appPoolName"){
 			$appPoolPid = $appPool.PID
 		}
 	}
 	write-host "$solutionFriendlyName app pool PID: $appPoolPID"
 	return $appPoolPid
 }
 
 #------------------------------------------------
 #------------------------------------------------
 function start-apppool{
 	$uri = new-object System.Uri("$wakeUpServiceUrl")
 	$client = new-object System.Net.WebClient
 	$client.DownloadString($uri) | out-null
 }
 
 #-------------------
 #-------------------
 function kill-apppool{
 	$procId = get-apppoolpid
 	if ($procId -ne 0){
 		write-host "Killing app pool process"
 		$proc = [System.Diagnostics.Process]::GetProcessById($procId)
 		$proc.Kill()      
 	}
 }
 
 #------------------------------------------------------------------------------
 #------------------------------------------------------------------------------
 function instrument2([string]$assemblyName, [string[]]$excludes){
 	$assy = "$websiteBinDirectory\$assemblyName"
 	$excludeLine = ""
 	if ($excludes -ne $null){
         foreach ($x in $excludes){
 		    write-host "Excluding: $x.*"	     
 		    $excludeLine = "$excludeLine-exclude:$x.* "
 	  }
 	}
     $cmd = "C:\Program Files\Microsoft Visual Studio 10.0\Team Tools\Performance Tools\vsinstr.exe"
     write-host "Executing $cmd /coverage $excludeLine $assy" 
 	& $cmd /coverage "$excludeLine" "$assy" 
 }
 
 #------------------------------------------------------------------------------
 #------------------------------------------------------------------------------
 function instrument([string]$assemblyName, [string[]]$excludes){
 	$assy = "$websiteBinDirectory\$assemblyName"
 
 	#$excludeParms = ""
 	#$args
 	#}
 	#$excludeParms
 	#& 'C:\Program Files\Microsoft Visual Studio 10.0\Team Tools\Performance Tools\vsinstr.exe' /coverage $excludeParms $assy 
 
 	if ($excludes -eq $null){
 		write-host "Excluding 0 funcspecs"
 		& 'C:\Program Files\Microsoft Visual Studio 10.0\Team Tools\Performance Tools\vsinstr.exe' /coverage $assy 
 	}
 	elseif ($excludes.Length -eq 0){
 		write-host "Excluding 0 funcspecs"
 		& 'C:\Program Files\Microsoft Visual Studio 10.0\Team Tools\Performance Tools\vsinstr.exe' /coverage $assy 
 	}
 	elseif ($excludes.Length -eq 1){
 		write-host "Excluding 1 funcspecs"
 		$exclude1 = '-exclude:' + $excludes[0] 
 		& 'C:\Program Files\Microsoft Visual Studio 10.0\Team Tools\Performance Tools\vsinstr.exe' /coverage $exclude1 $assy 
 	}
 	elseif ($excludes.Length -eq 2){
 		write-host "Excluding 2 funcspecs"
 		$exclude1 = '-exclude:' + $excludes[0]
 		$exclude2 = '-exclude:' + $excludes[1]
 		& 'C:\Program Files\Microsoft Visual Studio 10.0\Team Tools\Performance Tools\vsinstr.exe' /coverage $exclude1 $exclude2  $assy 
 	}
 	elseif ($excludes.Length -eq 3){
 		write-host "Excluding 3 funcspecs"
 		$exclude1 = '-exclude:' + $excludes[0]
 		$exclude2 = '-exclude:' + $excludes[1]
 		$exclude3 = '-exclude:' + $excludes[2]
 		& 'C:\Program Files\Microsoft Visual Studio 10.0\Team Tools\Performance Tools\vsinstr.exe' /coverage $exclude1 $exclude2 $exclude3 $assy 
 	}
 	elseif ($excludes.Length -eq 4){
 		write-host "Excluding 4 funcspecs"
 		$exclude1 = '-exclude:' + $excludes[0]
 		$exclude2 = '-exclude:' + $excludes[1]
 		$exclude3 = '-exclude:' + $excludes[2]
 		$exclude4 = '-exclude:' + $excludes[3]
 		& 'C:\Program Files\Microsoft Visual Studio 10.0\Team Tools\Performance Tools\vsinstr.exe' /coverage $exclude1 $exclude2 $exclude3 $exclude4 $assy 
 	}
 	elseif ($excludes.Length -eq 5){
 		write-host "Excluding 5 funcspecs"
 		$exclude1 = '-exclude:' + $excludes[0]
 		$exclude2 = '-exclude:' + $excludes[1]
 		$exclude3 = '-exclude:' + $excludes[2]
 		$exclude4 = '-exclude:' + $excludes[3]
 		$exclude5 = '-exclude:' + $excludes[4]		
 		& 'C:\Program Files\Microsoft Visual Studio 10.0\Team Tools\Performance Tools\vsinstr.exe' /coverage $exclude1 $exclude2 $exclude3 $exclude4 $exclude5 $assy 
 	}	
 	elseif ($excludes.Length -eq 6){
 		write-host "Excluding 6 funcspecs"
 		$exclude1 = '-exclude:' + $excludes[0]
 		$exclude2 = '-exclude:' + $excludes[1]
 		$exclude3 = '-exclude:' + $excludes[2]
 		$exclude4 = '-exclude:' + $excludes[3]
 		$exclude5 = '-exclude:' + $excludes[4]		
 		$exclude6 = '-exclude:' + $excludes[5]		
 		& 'C:\Program Files\Microsoft Visual Studio 10.0\Team Tools\Performance Tools\vsinstr.exe' /coverage $exclude1 $exclude2 $exclude3 $exclude4 $exclude5 $exclude6 $assy 
 	}
 }
 
 #-----------------------------------
 #------------------------------------
 function start-instrumentation{
 	&  'C:\Program Files\Microsoft Visual Studio 10.0\Team Tools\Performance Tools\vsperfclrenv' /globaltraceon 
 
 	kill-apppool
 	start-apppool
 	$procId = get-apppoolpid
 
 	& 'C:\Program Files\Microsoft Visual Studio 10.0\Team Tools\Performance Tools\vsperfcmd' /START:COVERAGE /OUTPUT:$coverageOutputPath /CS /USER:$appPoolIdentity
 	& 'C:\Program Files\Microsoft Visual Studio 10.0\Team Tools\Performance Tools\vsperfcmd' /ATTACH:$procId
 	& 'C:\Program Files\Microsoft Visual Studio 10.0\Team Tools\Performance Tools\vsperfcmd' /status
 }
 
 #--------------------
 #--------------------
 function stop-instrumentation{
 	& 'C:\Program Files\Microsoft Visual Studio 10.0\Team Tools\Performance Tools\vsperfcmd' /DETACH
 	kill-apppool
 	& 'C:\Program Files\Microsoft Visual Studio 10.0\Team Tools\Performance Tools\vsperfcmd' /SHUTDOWN 
 }
 
 #------------------------
 #------------------------
 function export-coverage{
 	[System.Reflection.Assembly]::LoadFile("C:\Program Files\Microsoft Visual Studio 10.0\Common7\IDE\PrivateAssemblies\Microsoft.VisualStudio.Coverage.Analysis.dll")
 	$coverage = [Microsoft.VisualStudio.Coverage.Analysis.CoverageInfo]::CreateFromFile("$coverageOutputPath")
 	$dataSet = $coverage.BuildDataSet()
 	$dataSet.WriteXml("$coverageOutputPath" + 'xml')
 }
 
 #----------------------------------------------------------------------------------
 #----------------------------------------------------------------------------------
 function delete-intstrumentedassemblies{
 	$dir = new-object IO.DirectoryInfo($websiteBinDirectory)
 	$originalAssemblies = $dir.GetFiles("*.dll.orig")
 	foreach ($originalAssembly in $originalAssemblies){
 		$targetName = $originalAssembly.FullName.TrimEnd(".orig".ToCharArray())
 		[IO.File]::Copy($originalAssembly.FullName, $targetName, $true)
 		[IO.File]::Delete($originalAssembly.FullName)
 		$instrumentedPdbName = $originalAssembly.FullName.TrimEnd(".dll.orig".ToCharArray()) + ".instr.pdb"
 		[IO.File]::Delete($instrumentedPdbName)
 	}
 }
 
 #--------------
 #--------------
 
 $solutionFriendlyName = 'XYZ'
 $wakeUpServiceUrl = 'http://localhost/x.y.z/Service.svc'
 $appPoolName = 'ap_XYZ'
 $appPoolIdentity = 'domain\svc_user'
 $websiteBinDirectory = 'c:\websites\xyz\x.y.z.Services\bin'
 $coverageOutputPath = "Test.coverage"
 
 instrument   "x.y.z.Services.dll"  
 instrument   "x.y.z.Core.dll"   'x.y.z.Core.Ignore1.*' , 'x.y.z.Core.Ignore2.*'  
 
 write-host "Before starting instrumentation, last exit code: $LASTEXITCODE"
 
 start-instrumentation
 
 write-host "Before running tests, last exit code: $LASTEXITCODE"
 & 'C:\Windows\Microsoft.NET\Framework\v4.0.30319\msbuild.exe' Build.proj /t:$msBuildTarget /p:ConfigurationName=$configurationName
 
 $realExitCode = $LASTEXITCODE
 
 write-host "Before stopping instrumentation, last exit code: $LASTEXITCODE, real exit code: $realExitCode"
 stop-instrumentation
 export-coverage
 export-coverage
 
 if ($deleteInstrumentedAssemblies -eq $true){
 	delete-intstrumentedassemblies
 }
 
 write-host "Before existing, last exit code: $LASTEXITCODE, real exit code: $realExitCode"
 exit $realExitCode
 
 #------------
 #------------
`

