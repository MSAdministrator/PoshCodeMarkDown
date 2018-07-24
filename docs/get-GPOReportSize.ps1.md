---
Author: chriskenis
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3980
Published Date: 2013-02-21t12
Archived Date: 2013-02-28t00
---

# get-gporeportsize - 

## Description

html report output for all (= default) or specific gpo (piped from get-gpo) with folder size

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 param(
 [Parameter(Position=0,ValueFromPipeline=$True)]$GPOs = @(Get-GPO -All),
 [string] $Reportfolder = [Environment]::getfolderpath("mydocuments") + "\GPOreports"
 )
 
 begin{
 $script:GPObj = @()
 $domain = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain().Name
 $startfolder = "\\$domain\SYSVOL\$domain\Policies"
 write-verbose "getting info from $startfolder"
 
 Function GPOSize($objGPO){
 	$colItems = (Get-ChildItem "$("$startfolder\`{")$($objGPO.Id)$("`}")" -recurse | Measure-Object -property length -sum)
 	$Result = "{0:N2}" -f ($colItems.sum / 1KB) + " KB"
 	write-verbose "the size for $objGPO is $Result"
 	return $Result
 }
 
 process{
 foreach ($GPO in $GPOs){
 	write-verbose "getting GPO $($GPO.DisplayName)"
 	$DateRevMin = get-date -uformat "%Y-%m-%d"
 	New-Item $Reportfolder\$DateRevMin -ItemType directory -Force | out-null
 	$OutPath = $Reportfolder + "\" + $DateRevMin + "\" + $GPO.DisplayName + ".html"
 	Get-GPOReport $GPO.Id -ReportType html -Path $OutPath
 	$script:GPObj += New-Object PSObject -Property @{
 		GPName = $GPO.DisplayName
 		GPGUID = $GPO.ID
 		Size = $(GPOSize $GPO)
 		Report = $OutPath
 		}
 	}
 
 end{
 $script:GPObj
 }
 
`

