---
Author: agnostracised
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4679
Published Date: 2016-12-09t16
Archived Date: 2016-11-01t13
---

# ad forest gpo auditing - 

## Description

this script queries the entire forest and a) dumps all gpo names with last modified and guid to \all_gpos and b) compares the current run

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 This script queries the entire forest and a) dumps all GPO names with last modified and GUID to \All_GPOs and b) Compares the current run
 against the last run and will list all modified GPOs in the \Modified_GPOs folder. The intention of this is to audit GPO modifications. Obviously the
 first run will not report any changes and simply dump all of the current GPOs.
 
 In order to populate the modified by inforamtion, directory services auditing must be enabled and the GPO containers auditted. More information
 on that can be found here: http://blogs.msdn.com/b/canberrapfe/archive/2012/05/02/auditing-group-policy-changes.aspx
 The account running the script needs to be able to read domain controller event logs as well.
 
 When the script looks for the modified by information in the event log, it will query every domain controller in that specific domain and look for
 the last event containing the GUID of the GPO. It then parses that information and grabs the modified information.
 #>
 Import-Module GroupPolicy
 Import-Module ActiveDirectory
 
 $colNewGPOs = @()
 $colAllNewGPOs = @()
 [System.DirectoryServices.ActiveDirectory.Forest]::GetCurrentForest().Domains | Select Name | ForEach {
 
 $DomainControllers = Get-ADDomainController -Server $_.Name -Filter *
 
 [DateTime]$Date = "01/01/1981"
 
 $count = $DomainControllers | measure-object
 $i = 0
 
 
     $Domain = $_.Name
 
     $FileName = $Domain + "_GPOs.csv"
     $FilePath = Split-Path -Parent -Path $MyInvocation.MyCommand.Definition
     If ((Test-Path ($FilePath + "\All_GPOs")) -eq $false){New-Item -ItemType Directory -Force -Path ($FilePath + "\All_GPOs")}
     If ((Test-Path ($FilePath + "\Modified_GPOs")) -eq $false){New-Item -ItemType Directory -Force -Path ($FilePath + "\Modified_GPOs")}
 
     $FilePath = $FilePath + "\All_GPOs\" + $FileName
    
     If (Test-Path $FilePath){$arrayOld = @(Import-CSV $FilePath)} Else {$arrayOld = @()}
     $arrayNew = @(Get-GPO -all -Domain $Domain | Select DisplayName, ModificationTime, Id | Sort-Object DisplayName)
     $arrayNew | Export-CSV $FilePath -notype
 
     ForEach($oldItem in $ArrayOld){
     $i = 0
         ForEach($newItem in $arrayNew){
                 $LoginID = "n/a"
                 $SourceDC = "n/a"
             If($oldItem.Id -ceq $newItem.Id){
                 $oldDate = Get-Date($oldItem.ModificationTime)
                 $newDate = Get-Date($newItem.ModificationTime)
                 If($oldDate -ne $newDate){
                 $LoginID = "n/a"
                 $SourceDC = "n/a"
                 $i = 0
                     $objNewGPOs = New-Object System.Object
                     $objNewGPOs | Add-Member -type NoteProperty -name Name -value $newItem.DisplayName
                     $objNewGPOs | Add-Member -type NoteProperty -name LastModified -value $newItem.ModificationTime
                     
                     
                     $start = $newItem.ModificationTime
                     $end = $start.addMinutes(10)
                     $start = $start.addMinutes(-10)
                     $start = Get-Date $start -format G
                     $end = Get-Date $end -format G
                     
                     Foreach($DC in $DomainControllers)
                     {
                     $DefaultPartition = $DC.DefaultPartition
                     $i++
                     Write-Progress -Activity "Querying Domain Controllers" -status $DC.HostName -percentComplete ($i / $count.Count*100)
                     
                     $GUID = [String]$newItem.Id
                     $GUID = "{" + $GUID + "}"
                     $DCHostname = $DC.HostName
                     write-host "$DefaultPartition : $GUID : $DCHostname"
                                         
                     Try{
                     
                     $content = Get-WinEvent -MaxEvents 1 -ComputerName $DC.HostName -FilterXML "<QueryList><Query Id=`"0`" Path=`"Security`"><Select Path=`"Security`">*[EventData[Data and (Data=`"CN=$GUID,CN=POLICIES,CN=SYSTEM,$DefaultPartition`")]] and *[System[(EventID=5136)]]</Select></Query></QueryList>" -ErrorAction Stop
                     $content
                     $eventXML = [XML]$content.ToXML()
 
                         $SourceDC = $DC.HostName
                         $SourceDC
                         $LoginID
                     
                      }
                      Catch{
                      $($_.Exception.Message)
                      }   
                     }
                     $objNewGPOs | Add-Member -type NoteProperty -name ModifiedBy -value $LoginID
                     $objNewGPOs | Add-Member -type NoteProperty -name SourceDC -value $SourceDC
                     $colNewGPOs += $objNewGPOs
                 }
             }
         }
     }
     
     $FileName = "All_Current_GPO_Modifications.csv"
     $FilePath = Split-Path -Parent -Path $MyInvocation.MyCommand.Definition
     $FilePath = $FilePath + "\Modified_GPOs\" + $FileName
     $colAllNewGPOs += $colNewGPOs
     $colAllNewGPOs | Sort-Object LastModified | Export-CSV $FilePath -notype
     
     $FileName = $Domain + "_GPO_Modifications.csv"
     $FilePath = Split-Path -Parent -Path $MyInvocation.MyCommand.Definition
     $FilePath = $FilePath + "\Modified_GPOs\" + $FileName
     If (Test-Path $FilePath){
         $arrayOldFile = @(Import-CSV $FilePath) | ForEach{
         $objNewGPOs = New-Object System.Object
         $objNewGPOs | Add-Member -type NoteProperty -name Name -value $_.Name
         $objNewGPOs | Add-Member -type NoteProperty -name LastModified -value $_.LastModified
         $objNewGPOs | Add-Member -type NoteProperty -name ModifiedBy -value $_.ModifiedBy
         $objNewGPOs | Add-Member -type NoteProperty -name SourceDC -value $_.SourceDC
         $colNewGPOs += $objNewGPOs
                     }
     } Else {$arrayOldFile = @()}
         
     $colNewGPOs | Sort-Object LastModified | Export-CSV $FilePath -notype
     $colNewGPOs = @()
 }
`

