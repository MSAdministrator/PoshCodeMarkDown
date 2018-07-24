---
Author: rov3_
Publisher: 
Copyright: 
Email: 
Version: 1.5
Encoding: ascii
License: cc0
PoshCode ID: 6584
Published Date: 2017-10-18t20
Archived Date: 2017-01-08t07
---

# wsus production approval - 

## Description

compare updates between wsus target groups (e.g test and production) and approve updates to the latter and email report on changes made.

## Comments



## Usage



## TODO



## function

`get-wsusgroupupdates`

## Code

`#
 #
 
 $date = Get-Date
 $targetGroup = "Enter the name of your production WSUS target group here"
 $updateFilter = "$_.title -notlike '*Server*' -and $_.title -notlike '*sharepoint*'"
 $reportSMTP = "your SMTP server"
 $reportTitle = "<h1>$env:computername</h1><h3>$updateCount updates approved for $targetGroup on $date.</h3>"
 $reportFrom = "address status email comes from"
 $reportTo = "address status email goes to"
 $reportSubject = "Updates approved for $targetGroup"
 
 
 
 $reportHeader = @"
     <link rel="stylesheet" src="https://necolas.github.io/normalize.css/latest/normalize.css">
     <style>
         body {
             font-family: sans-serif;
             font-size: 14px;
             margin: 2% 0;
         }
         h1 {
             font-size: 2em;
             font-weight: normal;
             padding: 0 2%;
         }
         h3 {
             font-size: 1.25em;
             font-weight: normal;
             padding: 0 2%;
         }
         table {
             border-collapse: collapse;
             width: 100%;
         }
         tr:nth-child(even) {
 		}
         tr:nth-child(odd) {
         }
         tr:first-child {
 		}
         th {
             font-weight: bold;
             text-align: left;
         }
         td,
         th {
             padding: .25em;
         }
         td:first-child,
         th:first-child    {
             padding-left: 2%;
         }
         td:last-child,
         th:last-child {
             padding-left: 2%;
         }
         h2 {
             font-size: 1.5em;
             font-weight: normal;
             margin: 1 0 .5%;
             padding: 0 2%;
         }
     </style>
 "@
 
 
 function Get-WSUSgroupupdates {
 Param ($target)
 
     [void][reflection.assembly]::LoadWithPartialName("Microsoft.UpdateServices.Administration")
     $wsus = [Microsoft.UpdateServices.Administration.AdminProxy]::getUpdateServer()
     $updateScope = new-object Microsoft.UpdateServices.Administration.UpdateScope;
     $updateScope.UpdateApprovalActions =[Microsoft.UpdateServices.Administration.UpdateApprovalActions]::Install -bor [Microsoft.UpdateServices.Administration.UpdateApprovalActions]::Uninstall -bor [Microsoft.UpdateServices.Administration.UpdateApprovalActions]:: All -bor [Microsoft.UpdateServices.Administration.UpdateApprovalActions]::NotApproved
      
     $updates = $wsus.GetUpdates($updateScope)
      
     $groups = $wsus.GetComputerTargetGroups() | where {$_.name -eq $target}
     $updateRep = @() 
     
     foreach($update in $updates) {
      
         foreach($group in $groups) {
         
         $status = "Install"
      
             if ($update.GetUpdateApprovals($group).Count -ne 0)
             {$status = $update.GetUpdateApprovals($group)[0].Action}
             
             $updateRep += $update
                 #@{Label='Title';Expression={$update.Title}},`
                 #@{Label='Group';Expression={$group.Name}},`
                 #@{Label='Status';Expression={$status}},`
                 #@{Label='guid';Expression={$update.id.updateid}}
             }
         }
 
         $updateRep
     }
 
 
 $testGroup = Get-WSUSgroupupdates -target "Workstations Test"
 $corpGroup = Get-WSUSgroupupdates -target "Production Corporate"
 
 
 Compare-Object $testGroup.id.updateid $corpGroup.id.updateid | `
     where {$_.SideIndicator -eq '<='} | `
     ForEach-Object  { $findDiff += $_.InputObject }
 
 
 $allUpdate = $findDiff | where {$updateFilter} | `
     Approve-WsusUpdate -Action Install -TargetGroupName $targetGroup
 
 
 $updateCount = ($allUpdate.update.title).Count | Out-String
 
 
 $outBody = $allUpdate.Update | select `
     @{Label='Update';Expression={$_.title}},`
     @{Label='Released';Expression={$_.creationdate}},`
     @{Label='Type';Expression={$_.UpdateClassificationTitle}} | `
     ConvertTo-Html -fragment -Property Update,Released,Type -PreContent $reportTitle
 
 
 $outHTM = ConvertTo-Html -Head $reportHeader -Body $outBody
 
 Send-MailMessage `
    -SmtpServer $reportSMTP `
    -BodyAsHtml ($outHTM | out-string) `
    -From $reportFrom `
    -To $reportTo `
    -Subject $reportSubject
`

