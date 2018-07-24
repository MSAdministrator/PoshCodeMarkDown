---
Author: mostafa elhemali
Publisher: 
Copyright: 
Email: 
Version: 9600.6
Encoding: ascii
License: cc0
PoshCode ID: 5066
Published Date: 2015-04-09t17
Archived Date: 2015-01-28t02
---

# hadoop-dev - 

## Description

script to setup a working environment for working with apache hadoop code on windows.

## Comments



## Usage



## TODO



## function

`get-scriptdirectory`

## Code

`#
 #
 <#
 .SYNOPSIS
 Prepares your dev environment for working with Apache Hadoop.
 #>
 
 function Get-ScriptDirectory
 {
     $Invocation = (Get-Variable MyInvocation -Scope 1).Value
     Split-Path $Invocation.MyCommand.Path
 }
 
 $sourceDirectory = Split-Path $(Get-ScriptDirectory)
 $sourceCodeDirectory = "$sourceDirectory\hadoop-common"
 $toolsDirectory = "$sourceDirectory\Tools"
 $distDirectory = "$sourceDirectory\hadoop-common\hadoop-dist\target\hadoop-3.0.0-SNAPSHOT"
 $defaultSingleNodeDirectory = 'c:\YarnSingleNode'
 $defaultDownloadDirectory = "$sourceDirectory\Downloads"
 $defaultLogsDirectory = "$sourceDirectory\Logs"
 $defaultPatchesDirectory = "$sourceDirectory\Patches"
 
 function Unzip($fileName, $destination)
 {
     $shellApp = New-Object -com shell.application
     $zipFile = $shellApp.namespace($fileName)
     $mdOutput = md -Force $destination
     $destinationShell = $shellApp.namespace($destination)
     $firstItem = $zipFile.items() | Select-Object -first 1 | %{Split-Path -Leaf $_.Path}
     if ($(Test-Path $(Join-Path $destination $firstItem)))
     {
         Write-Host 'Unzip destination already exists - skipping...'
     }
     else
     {
         $destinationShell.CopyHere($zipFile.items())
     }
     return Join-Path $destination $firstItem
 }
 
 function Get-Ant($downloadsDirectory = $defaultDownloadDirectory, $logsDirectory = $defaultLogsDirectory, $installDirectory = $toolsDirectory)
 {
     $wc = New-Object System.Net.WebClient
     $baseUrl = 'http://www.apache.org/dist/ant/binaries/'
     $binariesPage = $wc.DownloadString($baseUrl)
     $regex = [regex]'href="(?<URL>.*\.zip)"'
     $zipName = $($regex.Matches($binariesPage) | %{$_.Groups['URL'].Value})
     $downloadUrl = $baseUrl + $zipName
     $destZip = Join-Path $downloadsDirectory $zipName
     if (-not $(Test-Path $destZip))
     {
         Write-Host "Downloading Ant..."
         $wc.DownloadFile($downloadUrl, $destZip)
         Write-Host "Done!"
     }
     else
     {
         Write-Host 'Ant zip file already present - using that.'
     }
     Write-Host "Extracting Ant..."
     Unzip $destZip $installDirectory
     Write-Host "Done!"
 }
 
 function Get-Maven($downloadsDirectory = $defaultDownloadDirectory, $logsDirectory = $defaultLogsDirectory, $installDirectory = $toolsDirectory)
 {
     $wc = New-Object System.Net.WebClient
     $baseUrl = 'http://www.apache.org/dist/maven/binaries/'
     $binariesPage = $wc.DownloadString($baseUrl)
     $regex = [regex]'href="(?<URL>apache-maven-(?<Ver>[0-9.]*)-bin\.zip)"'
     $zipName = $regex.Matches($binariesPage) | Sort-Object -property @{Expression={[System.Version]$_.Groups['Ver'].Value};Ascending=$false} | Select-Object -First 1 | %{$_.Groups['URL'].Value}
     $downloadUrl = $baseUrl + $zipName
     $destZip = Join-Path $downloadsDirectory $zipName
     if (-not $(Test-Path $destZip))
     {
         Write-Host "Downloading Maven..."
         $wc.DownloadFile($downloadUrl, $destZip)
         Write-Host "Done!"
     }
     else
     {
         Write-Host 'Maven zip file already present - using that.'
     }
     Write-Host "Extracting Maven..."
     Unzip $destZip $installDirectory
     Write-Host "Done!"
 }
 
 function Get-ProtoBuf($downloadsDirectory = $defaultDownloadDirectory, $logsDirectory = $defaultLogsDirectory, $installDirectory = $toolsDirectory)
 {
     $wc = New-Object System.Net.WebClient
     $downloadUrl = 'https://protobuf.googlecode.com/files/protoc-2.5.0-win32.zip'
     $zipName = 'protoc-2.5.0-win32.zip'
     $destZip = Join-Path $downloadsDirectory $zipName
     if (-not $(Test-Path $destZip))
     {
         Write-Host "Downloading ProtoBuf..."
         $wc.DownloadFile($downloadUrl, $destZip)
         Write-Host "Done!"
     }
     else
     {
         Write-Host 'ProtoBuf zip file already present - using that.'
     }
     $destination = Join-Path $installDirectory 'ProtoBuf'
     Write-Host "Extracting ProtoBuf..."
     $protoc = Unzip $destZip $destination
     Write-Host "Done!"
     $destination
 }
 
 function Apply-SpacePatch($patchesDirectory = $defaultPatchesDirectory, $dest = $sourceCodeDirectory, $logsDirectory = $defaultLogsDirectory)
 {
     Trap { "Exception applying space patch`: $_"; Break }
     $wc = New-Object System.Net.WebClient
     $downloadUrl = 'http://issues.apache.org/jira/secure/attachment/12639264/HADOOP-9600.6.patch'
     $mdOutput = md -Force $patchesDirectory
 	$localPath = "$patchesDirectory\SpacePatch.patch"
     $wc.DownloadFile($downloadUrl, $localPath)
 	pushd $dest
 	git apply $localPath > $(Join-Path $logsDirectory 'GitApply.log') 2>&1
 	popd
 }
 
 function Add-Path($directory)
 {
     if (-not($env:PATH -like "*$directory*"))
     {
         $env:PATH = "$env:PATH;$directory;"
     }
 }
 
 function Get-Trunk($dest = $sourceDirectory, $logsDirectory = $defaultLogsDirectory)
 {
     Write-Host 'Downloading Hadoop trunk source'
     pushd $dest
     git clone https://github.com/apache/hadoop-common.git > $(Join-Path $logsDirectory 'GitCheckout.log') 2>&1
     popd
     Write-Host 'Done!'
 }
 
 $antDirectory = "$toolsDirectory\apache-ant-1.9.1\bin"
 $mavenDirectory = "$toolsDirectory\apache-maven-3.2.1\bin"
 $protoBufDirectory = "$toolsDirectory\ProtoBuf"
 Add-Path $antDirectory
 Add-Path $mavenDirectory
 Add-Path $protoBufDirectory
 Add-Path "$toolsDirectory\cygwin\bin"
 Add-Path "$env:windir\Microsoft.NET\Framework64\v4.0.30319"
 Add-Path "$env:ProgramFiles\Java\bin"
 Add-Path "$env:ProgramFiles\Java\jre\bin"
 $env:JAVA_HOME = "$env:ProgramFiles\Java\jdk1.7.0_51"
 $env:PLATFORM = 'x64'
 
 $scriptLogsDirectory = "$env:LOCALAPPDATA\HadoopDevScriptLogs"
 $mdOutput = md -Force $scriptLogsDirectory
 $mdOutput = md -Force $defaultLogsDirectory
 $mdOutput = md -Force $defaultDownloadDirectory
 
 function Check-Dist
 {
     if (-not (Test-Path $distDirectory))
     {
         Write-Error 'Build not found, please run: Build-Package'
         return $false;
     }
     return $true;
 }
 
 function Build-Package([Switch]$noClean = $false)
 {
     pushd $sourceDirectory\hadoop-common
     mvn $(if ($noClean) {""} else {"clean"}) package -DskipTests -Pdist -Dtar | Tee-Object -FilePath BuildLogs.txt
     popd
 }
 
 function Alter-XmlFile
 (
     [Parameter(Mandatory = $true, HelpMessage = 'The file to alter.')]
     [string]$xmlFile,
     [Parameter(Mandatory = $true, HelpMessage = 'The new XML for the file')]
     [xml]$xml
 )
 {
     Trap { "Exception altering file $($xmlFile)`: $_"; Break }
 
     $xmlFile = (Resolve-Path $xmlFile).Path
     $reader = New-Object System.Xml.XmlTextReader $xmlFile
     $didRead = $reader.Read()
     $encoding = $reader.Encoding
     $reader.Close()
     $writerSettings = New-Object System.Xml.XmlWriterSettings
     $writerSettings.Encoding = $encoding
     $writerSettings.Indent = $true
     $writerSettings.IndentChars = "  "
     $writer = [System.Xml.XmlWriter]::Create($xmlFile, $writerSettings)
     $xml.WriteContentTo($writer)
     $writer.Close()
 }
 
 function Add-Property($xmlFile, $propertyName, $propertyValue)
 {
     if (-not (Test-Path $xmlFile))
     {
         Out-File -FilePath $xmlFile -Encoding ascii -InputObject @"
 <?xml version="1.0" encoding="UTF-8"?>
 <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
 <configuration>
 
 </configuration>
 "@
     }
     [xml]$xml = Get-Content $xmlFile
     $propertyNode = $xml.CreateElement('property')
     $nameNode = $xml.CreateElement('name')
     $nameNode.InnerText = $propertyName
     $inserted = $propertyNode.InsertAfter($nameNode, $null)
     $valueNode = $xml.CreateElement('value')
     $valueNode.InnerText = $propertyValue
     $inserted = $propertyNode.InsertAfter($valueNode, $nameNode)
     $inserted = $xml.DocumentElement.InsertAfter($propertyNode, $xml.DocumentElement.LastChild)
     Alter-XmlFile $xmlFile $xml
 }
 
 function Configure-SingleNode($destinationDirectory = $defaultSingleNodeDirectory)
 {
     Trap
     {
         $_
         break;
     }
     if (Check-Dist)
     {
         Write-Host "Copying Hadoop..."
         $robocopyOutput = robocopy /MIR $distDirectory $destinationDirectory
 		$mdOutput = md "$destinationDirectory\logs"
         
         $clusterFilesDirectory = "$destinationDirectory\ClusterFiles"
         $clusterFilesUri = $(New-Object -Type 'System.Uri' -ArgumentList $clusterFilesDirectory).AbsoluteUri
         
         Write-Host "Writing out configuration files..."
         $mapredSite = "$destinationDirectory\etc\hadoop\mapred-site.xml"
         Add-Property $mapredSite 'mapreduce.framework.name' "yarn"
         Add-Property $mapredSite 'fs.defaultFS' "hdfs://$($localHost):9000"
         
         $yarnSite = "$destinationDirectory\etc\hadoop\yarn-site.xml"
         Add-Property $yarnSite 'yarn.resourcemanager.resource-tracker.address' "$($localHost):6010"
         Add-Property $yarnSite 'yarn.resourcemanager.scheduler.address' "$($localHost):6011"
         Add-Property $yarnSite 'yarn.resourcemanager.scheduler.class' 'org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler'
         Add-Property $yarnSite 'yarn.resourcemanager.address' "$($localHost):6012"
         Add-Property $yarnSite 'yarn.nodemanager.local-dirs' "$clusterFilesDirectory\NMLocal"
         Add-Property $yarnSite 'yarn.nodemanager.address' "$($localHost):6013"
         Add-Property $yarnSite 'yarn.nodemanager.log-dirs' "$clusterFilesDirectory\NMLogs"
         Add-Property $yarnSite 'yarn.nodemanager.aux-services' 'mapreduce_shuffle'
         
         $hdfsSite = "$destinationDirectory\etc\hadoop\hdfs-site.xml"
         Add-Property $hdfsSite 'fs.defaultFS' "hdfs://$($localHost):9000"
         Add-Property $hdfsSite 'dfs.replication' '1'
         Add-Property $hdfsSite 'dfs.name.dir' "$clusterFilesUri/nn"
         Add-Property $hdfsSite 'dfs.data.dir' "$clusterFilesUri/dn"
         
         Write-Host "Done!"
     }
 }
 
 function Run-PowerShellSeparateWindow($command, $title)
 {
 	Start-Process powershell "`$(Get-Host).UI.RawUI.WindowTitle = '$title';$command"
 }
 
 function Run-HadoopNode($cmd, $logFileName, $title, $hadoopDirectory = $defaultSingleNodeDirectory)
 {
     Run-PowerShellSeparateWindow "cd `"$hadoopDirectory`";`$(Get-Host).PrivateData.ErrorForegroundColor = 'White'; $cmd 2>&1 | Tee-Object `"$hadoopDirectory/logs/$logFileName.txt`"" "$title"
 }
 
 function Run-NameNode($hadoopDirectory = $defaultSingleNodeDirectory)
 {
 	Run-HadoopNode "./bin/hdfs namenode" "NameNodeLogs" "Namenode"
 }
 
 function Run-DataNode($hadoopDirectory = $defaultSingleNodeDirectory)
 {
 	Run-HadoopNode "./bin/hdfs datanode" "DataNodeLogs" "Datanode"
 }
 
 function Run-ResourceManager($hadoopDirectory = $defaultSingleNodeDirectory)
 {
 	Run-HadoopNode "./bin/yarn resourcemanager" "ResourceManagerLogs" "ResourceManager"
 }
 
 function Run-NodeManager($hadoopDirectory = $defaultSingleNodeDirectory)
 {
 	Run-HadoopNode "./bin/yarn nodemanager" "NodeManagerLogs" "NodeManager"
 }
 
 function Run-SingleNode($singleNodeDirectory = $defaultSingleNodeDirectory, [Switch]$format = $(-not (Test-Path "$singleNodeDirectory\ClusterFiles\nn")))
 {
     if ($format)
     {
         pushd "$singleNodeDirectory";
 		./bin/hdfs namenode -format
 		popd
 
         $dataNodeDirectory = "$singleNodeDirectory\ClusterFiles\dn"
         if (Test-Path $dataNodeDirectory)
         {
             Remove-Item $dataNodeDirectory -Recurse
         }
     }
 
     Run-NameNode $singleNodeDirectory
     Run-DataNode $singleNodeDirectory
     Run-ResourceManager $singleNodeDirectory
     Run-NodeManager $singleNodeDirectory
 }
 
 function Run-Hadoop
 (
     [Parameter(Mandatory=$true,Position=0,ValueFromRemainingArguments=$true)]
     [String[]]
     $args,
     [Parameter()]
     $hadoopDirectory = $defaultSingleNodeDirectory
 )
 {
     pushd "$hadoopDirectory";
 	./bin/hadoop $args
 	popd
 }
 
 function Run-Hdfs
 (
     [Parameter(Mandatory=$true,Position=0,ValueFromRemainingArguments=$true)]
     [String[]]
     $args,
     [Parameter()]
     $hadoopDirectory = $defaultSingleNodeDirectory
 )
 {
     pushd "$hadoopDirectory";
 	./bin/hdfs $args
 	popd
 }
 
 function Invoke-Environment($Command)
 {
 	cmd /c "$Command > nul 2>&1 && set" | .{process{
 		if ($_ -match '^([^=]+)=(.*)') {
 			[System.Environment]::SetEnvironmentVariable($matches[1], $matches[2])
 		}
 	}}
 }
 
 Invoke-Environment "`"$env:ProgramFiles\Microsoft SDKs\Windows\v7.1\Bin\SetEnv.Cmd`" /x64 /Release"
 $(Get-Host).UI.RawUI.ForegroundColor = 'White'
`

