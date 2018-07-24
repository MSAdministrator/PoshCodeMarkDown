---
Author: chriskenis
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4052
Published Date: 2013-03-27t19
Archived Date: 2013-03-30t05
---

# getset-users - 

## Description

enumerate + add + remove users or groups to any local group on a local, single or multiple remote computers

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 param(
 [Parameter(Position=0,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
 [alias("Name","ComputerName")][string[]] $Computers = @($env:computername),
 [string] $NTDomain = ($env:UserDomain),
 [string[]] $LocalGroups = @("Administrators"),
 )
 begin{
 $script:objReport = @()
 
 Function ListMembers ($Group){
 $members = $Group.psbase.invoke("Members") | %{$_.GetType().InvokeMember("Name", 'GetProperty', $null, $_, $null)}
 return $members
 }
 #
 Function LocalNTGroup ($ComputerName,$Group){
 $NTgroup = ([ADSI]("WinNT://" + $Computer + ",computer")).psbase.children.find($Group)
 return $NTgroup
 }
 Function Output($ComputerName,$GroupName,$UserName,$Action,$Result){
 $script:objReport += New-Object PSObject -Property @{
 	Computername = $ComputerName
 	GroupName = $GroupName
 	UserName = $UserName
 	Action = $Action
 	Result = $Result
 	}
 write-host "on $ComputerName -- $GroupName for $UserName the result for $Action is: $Result"
 }
 
 process{
 foreach ($Computer in $Computers){
 	if (Test-Connection -ComputerName $Computer -Count 1 -Quiet -EA "Stop"){
 		write-verbose "getting local group members on $Computer"
 		foreach ($Group in $LocalGroups){
 			$LocalGroup = LocalNTGroup $Computer $Group
 			foreach ($member in $(ListMembers $LocalGroup)){
 				Output $Computer $Group $member "ENUM" "Existing"
 				}
 			foreach ($AddID in $Add){
 				try{
 					$LocalGroup.Add("WinNT://" + $NTDomain + "/" + $AddID)
 					$Action = "Added"
 					}
 				catch{
 					write-debug "$AddID is not added"
 					$Action = $Error[0].Exception.InnerException.Message.ToString().Trim()
 					}
 				Output $Computer $Group $AddID "ADDING" $Action
 				}
 			foreach ($RemID in $Remove){
 				try{
 					$LocalGroup.Remove("WinNT://" + $NTDomain + "/" + $RemID)
 					$Action = "Removed"
 					}
 				catch{
 					write-debug "$RemID is not added"
 					$Action = $Error[0].Exception.InnerException.Message.ToString().Trim()
 					}
 				Output $Computer $Group $RemID "REMOVING" $Action
 				}
 			}
 		}
 	}
 
 end{
 $script:objReport | select computername, groupname, username, action, result | sort username
 }
`

