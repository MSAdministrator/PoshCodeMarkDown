---
Author: jan egil ring
Publisher: 
Copyright: 
Email: 
Version: 1.1
Encoding: ascii
License: cc0
PoshCode ID: 2048
Published Date: 2010-08-04t05
Archived Date: 2016-11-01t14
---

# migrate-admtuser - migrate-admtuser.ps1

## Description

a function to migrate a single user in the active directory migration tool, based on the sample script invoke-admtusermigration.ps1

## Comments



## Usage



## TODO



## function

`migrate-admtuser`

## Code

`#
 #
 ###########################################################################"
 #
 #
 #
 #
 #
 #
 #
 #
 ###########################################################################"
 
 function Migrate-ADMTUser {
 <#
 .SYNOPSIS
 Migrate a single user object using ADMT
 .DESCRIPTION
 Migrates the specified source domain user object to the target domain.
 One mandatory parameter: samaccountname
 .PARAMETER samaccountname
 The samaccountname of the source domain user object to migrate
 .EXAMPLE
 Migrate-ADMTUser -samaccountname JDoe
 .NOTES
 AUTHOR:    Jan Egil Ring
 BLOG:      http://blog.powershell.no
 LASTEDIT:  04.08.2010
 #>
 
 [CmdletBinding()]
     param (
         [parameter(Mandatory=$true,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
         [string]$samaccountname
     )
 
 
 
 
 $admtComplexPassword                   = "&H0001"
 $admtCopyPassword                      = "&H0002"
 $admtDoNotUpdatePasswordsForExisting   = "&H0010"
 
 
 $admtIgnoreConflicting           = "&H0000"
 $admtMergeConflicting            = "&H0001"
 $admtRemoveExistingUserRights    = "&H0010"
 $admtRemoveExistingMembers       = "&H0020"
 $admtMoveMergedAccounts          = "&H0040"
 
 
 $admtLeaveSource        = "&H0000"
 $admtDisableSource      = "&H0001"
 $admtTargetSameAsSource = "&H0000"
 $admtDisableTarget      = "&H0010"
 $admtEnableTarget       = "&H0020"
 
 
 $admtNoExpiration = "-1"
 
 
 $admtTranslateReplace = "0"
 $admtTranslateAdd     = "1"
 $admtTranslateRemove  = "2"
 
 
 $admtReportMigratedAccounts  = "0"
 $admtReportMigratedComputers = "1"
 $admtReportExpiredComputers  = "2"
 $admtReportAccountReferences = "3"
 $admtReportNameConflicts     = "4"
 
 
 $admtNone     = "0"
 $admtData     = "1"
 $admtFile     = "2"
 $admtDomain   = "3"
 $admtRecurse           = "&H0100"
 $admtFlattenHierarchy  = "&H0000"
 $admtMaintainHierarchy = "&H0200"
 
 
 $admtProgressObjectMigration   = "0"
 $admtProgressAgentDispatch     = "1"
 $admtProgressAgentOperation    = "2"
 $admtProgressMailboxMigration  = "3"
 
 $admtEventNone                         = "0"
 $admtEventObjectInputPreprocessed      = "1"
 $admtEventTaskStart                    = "2"
 $admtEventTaskFinish                   = "3"
 $admtEventObjectProcessed              = "4"
 $admtEventGroupMembershipProcessed     = "5"
 $admtEventAgentStatusUpdate          ="6"
 $admtEventAgentNotStarted = "7"
 $admtEventAgentFailedToStart = "8"
 $admtEventAgentWaitingForReboot = "9"
 $admtEventAgentRunning = "10"
 $admtEventAgentCancelled = "11"
 $admtEventAgentPassed = "12"
 $admtEventAgentFailed = "13"
 $admtEventAgentWaitingForRetry = "14"
 $admtEventAgentSuccessful = "15"
 $admtEventAgentCompletedWithWarnings = "16"
 $admtEventAgentCompletedWithErrors = "17"
 $admtEventTaskLogSaved = "18"
 
 $admtAgentPreCheckPhase = "&H100"
 $admtAgentAgentOperationPhase = "&H200"
 $admtAgentPostCheckPhase = "&H300"
 
 $admtAgentStatusMask = "&HFF"
 $admtAgentPhaseMask = "&H300"
 
 $admtStatusSuccess   = "0"
 $admtStatusWarnings  = "1"
 $admtStatusErrors    = "2"
 
 
 $Migration = New-Object -ComObject "ADMT.Migration"
 $Migration.IntraForest = $false
 $Migration.SourceDomain = "contoso-old.local"
 $Migration.SourceDomainController = "dc01.contoso-old.local"
 $Migration.SourceOU = "Europe/Norway/Oslo"
 $Migration.TargetDomain = "contoso-new.local"
 $Migration.TargetDomainController = "dc02.contoso-new.local"
 $Migration.TargetOU = "Europe/Norway/Oslo"
 $Migration.PasswordOption = $admtComplexPassword
 $Migration.PasswordServer = "dc01.contoso-old.local"
 $Migration.PasswordFile = "C:\WINDOWS\ADMT\Logs\Passwords.txt"
 $Migration.ConflictOptions = $admtIgnoreConflicting
 $Migration.UserPropertiesToExclude = ""
 $Migration.InetOrgPersonPropertiesToExclude = ""
 $Migration.GroupPropertiesToExclude = ""
 $Migration.ComputerPropertiesToExclude = ""
 $Migration.SystemPropertiesToExclude = ""
 $Migration.PerformPreCheckOnly = $false
 $Migration.AdmtEventLevel = $admtStatusWarnings
 $Migration.CommandLine = $false
 
 $UserMigration = $Migration.CreateUserMigration()
 $UserMigration.DisableOption = $admtTargetSameAsSource
 $UserMigration.SourceExpiration = $admtNoExpiration
 $UserMigration.MigrateSIDs = $true
 $UserMigration.TranslateRoamingProfile = $false
 $UserMigration.UpdateUserRights = $false
 $UserMigration.MigrateGroups = $false
 $UserMigration.UpdatePreviouslyMigratedObjects = $false
 $UserMigration.FixGroupMembership = $true
 $UserMigration.MigrateServiceAccounts = $false
 
 $UserMigration.Migrate($admtData,$samaccountname,$null)
 
 $PasswordMigration = $Migration.CreatePasswordMigration()
 
 $PasswordMigration.Migrate($admtData,$samaccountname,$null)
 
 
 }
`

