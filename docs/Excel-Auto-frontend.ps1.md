---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1253
Published Date: 
Archived Date: 2009-09-12t11
---

# excel auto-frontend - 

## Description

automatically create an excel frontend to a cmdlet or advanced function. not really general purpose yet. there are a couple of examples within the script.

## Comments



## Usage



## TODO



## function

`new-dummyvm`

## Code

`#
 #
 Function New-DummyVM {
 	param(
 	[Parameter(Mandatory=$true,HelpMessage="Target Host")]
 	[VMware.VimAutomation.Types.VMHost]
 	$TargetHost,
 
 	[Parameter(Mandatory=$true,HelpMessage="Target Host")]
 	[VMware.VimAutomation.Types.ResourcePool]
 	$ResourcePool,
 
 	[Parameter(Mandatory=$true,HelpMessage="New VM Name")]
 	[String]
 	$NewName
 	)
 	
 	Get-VMHost $TargetHost | New-VM -ResourcePool $ResourcePool -Name $NewName -DiskMB 1
 }
 
 Function Add-BulkHosts {
 	param(
 	[Parameter(Mandatory=$true,HelpMessage="Host name or IP address")]
 	[String]
 	$HostName,
 
 	[Parameter(HelpMessage="Target Datacenter")]
 	[VMware.VimAutomation.Types.Datacenter]
 	$Datacenter,
 
 	[Parameter(HelpMessage="Target Folder")]
 	[VMware.VimAutomation.Types.Folder]
 	$Folder,
 
 	[Parameter(HelpMessage="Target Cluster")]
 	[VMware.VimAutomation.Types.Cluster]
 	$Cluster,
 
 	[Parameter(HelpMessage="Target Resource Pool")]
 	[VMware.VimAutomation.Types.ResourcePool]
 	$ResourcePool,
 
 	[Parameter(HelpMessage="Target Port")]
 	[Int32]
 	$Port=443,
 
 	[Parameter(HelpMessage="User")]
 	[String]
 	$User = "",
 
 	[Parameter(HelpMessage="Password")]
 	[String]
 	$Password
 	)
 
 	Process {
 		if ($Datacenter) {
 			$location = $Datacenter
 		} elseif ($Folder) {
 			$location = $Folder
 		} elseif ($Cluster) {
 			$location = $Cluster
 		} elseif ($ResourcePool) {
 			$location = $ResourcePool
 		}
 		if (!$location) {
 			Write-Warning "One of Datacenter, Folder, Cluster or ResourcePool must be specified."
 			return
 		}
 
 		if ($User -ne "") {
 			Add-VMHost -Name $HostName -Location $location -Port $Port -User $User -Password $Password -Force
 		} else {
 			Add-VMHost -Name $HostName -Location $location -Port $Port -Force
 		}
 	}
 }
 
 Function Get-ExcelFrontEnd {
 	param($cmdlet)
 
 	[reflection.assembly]::loadwithpartialname("microsoft.office.interop.excel") | Out-Null
 	$xlDvType                  = "Microsoft.Office.Interop.Excel.XlDVType" -as [type]
 	$xlDvAlertStyle            = "Microsoft.Office.Interop.Excel.XlDvAlertStyle" -as [type]
 	$xlFormatConditionOperator = "Microsoft.Office.Interop.Excel.XlFormatConditionOperator" -as [type]
 	$myDocuments = (Get-Itemproperty "hkcu:\Software\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders").Personal
 	$csvFile = $myDocuments + "\parameters.csv"
 
 	$startWriteTime = (Get-ChildItem $csvFile -ea SilentlyContinue).LastWriteTime
 
 	$findObjects = @{
 		[VMware.VimAutomation.Types.Cluster]             = { Get-Cluster }             ;
 		[VMware.VimAutomation.Types.Datacenter]          = { Get-Datacenter }          ;
 		[Datastore]                                      = { Get-Datastore }           ;
 		[VMware.VimAutomation.Types.Folder]              = { Get-Folder }              ;
 		[VMware.VimAutomation.Types.OSCustomizationSpec] = { Get-OSCustomizationSpec } ;
 		[VMware.VimAutomation.Types.ResourcePool]        = { Get-ResourcePool }        ;
 		[VMware.VimAutomation.Types.Template]            = { Get-Template }            ;
 		[VMware.VimAutomation.Types.VirtualMachine]      = { Get-VM }                  ;
 		[VMware.VimAutomation.Types.VMHost]              = { Get-VMHost }
 	}
 	
 	$ignore = @("Verbose", "Debug", "ErrorAction", "WarningAction", "ErrorVariable",
 	    "WarningVariable", "OutVariable", "OutBuffer")
 
 	$parameters = (Get-Command $cmdlet).Parameters
 
 	$cmdletTemplate = $myDocuments + "\CmdletFrontend.xlsm"
 	$ifOffset = 11
 
 	$xl = New-Object -ComObject Excel.Application
 	$wb = $xl.Workbooks.Open($cmdletTemplate)
 	#$wb = $xl.Workbooks.Add()
 	$ifSheet = $wb.worksheets.item(1)
 	#$ifSheet.Name = "Interface"
 	$ifSheet.StandardWidth = 20
 	$dataSheet = $wb.worksheets.item(2)
 	#$dataSheet.Name = "Data"
 	#$wb.worksheets.item(3).Delete()
 
 	$ifSheet.Cells.Item(4, 1) = $cmdlet
 	#$ifSheet.Cells.Item(7, 5) = $defaultVIServer.Name
 	#$ifSheet.Cells.Item(8, 5) = $defaultVIServer.SessionId
 
 	$xl.visible = $true
 
 	$i = 0
 	foreach ($key in $parameters.keys) {
 		if ($ignore -contains $key) {
 			continue
 		}
 		$i++
 
 		$parameter = $parameters[$key]
 		$type = $parameter.ParameterType
 
 
 		$ifSheet.Cells.Item($ifOffset, $i) = $key
 
 		if ($findObjects[$type]) {
 			$values = . $findObjects[$type]
 
 			$j = 1
 			foreach ($value in $values) {
 				$dataSheet.Cells.Item($j++, $i) = $value.Name
 			}
 
 			$rangeName = $key
 			Write-Debug $rangeName
 
 			$dataRange = "=Data!R1C{0}:R{1}C{2}" -f $i, ($j-1), $i
 			Write-Debug $dataRange
 			$name = $wb.names.add($rangename, $null, $true, $null, $null, $null, $null, $null, $null, $dataRange, $null)
 
 			$column = [char]([byte][char]"A" + $i - 1)
 			$ifRange = "{0}{1}:{2}{3}" -f $column, $ifOffset, $column, 250
 			Write-Debug $ifRange
 			$range = $ifSheet.Range($ifRange).Select()
 			$validation = $xl.Selection.Validation.Add($xlDvType::xlValidateList, $xlDvAlertStyle::xlValidAlertStop,
 				$xlFormatConditionOperator::xlBetween, "=$rangename", $null)
 		}
 	}
 	
 	$range = $ifSheet.Range("A1").Select()
 
 	do {
 		Write-Debug "Waiting"
 		Start-Sleep 1
 		$thisWriteTime = (Get-ChildItem $csvFile -ea silentlycontinue).LastWriteTime
 	} while ($thisWriteTime -le $startWriteTime)
 
 	$xl.Visible = $false
 	$wb.Close($false)
 	$xl.Quit()
 	foreach ($f in (import-csv $csvFile)) {
 		$properties = $f | Get-Member -type NoteProperty
  		$argString = ""
 		$objects = @()
 		$i = 0
 		foreach ($property in $properties) {
 			$pName = $property.Name
 			if ($f.$pName -ne "") {
 
 				$parameter = $parameters[$pName]
 				$type = $parameter.ParameterType
 				$getter = $findObjects[$type]
 				$object = $null
 				if ($getter) {
 					$lookupExpression = ($getter.ToString() + " -name '" + $f.$pName + "' | Select -first 1")
 					$object = Invoke-Expression $lookupExpression
 				} else {
 					$object = $f.$pName
 				}
 				$objects += $object
 				$argString += (" -" + $pName + " `$objects[$i]")
 				$i++
 			}
 		}
 		$command = $cmdlet + $argString
 		Invoke-Expression $command
 	}
 }
 
`

