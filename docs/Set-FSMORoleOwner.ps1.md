---
Author: boe prox
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2728
Published Date: 2011-06-10t10
Archived Date: 2011-08-14t09
---

# set-fsmoroleowner - 

## Description

this advanced function allows you to transfer or seize the fsmo roles to a specified domain controller. has support for -whatif and -confirm to avoid making a mistake with transferring the roles. also can display the current owners after the action has completed using -passthru.

## Comments



## Usage



## TODO



## function

`set-fsmoroleowner`

## Code

`#
 #
 Function Set-FSMORoleOwner {
 .SYNOPSIS  
     Performs a transfer of a FSMO role to a specified Domain Controller. 
     
 .DESCRIPTION  
     Performs a transfer of a FSMO role to a specified Domain Controller.
 
 .PARAMETER DomainController
     Fully Qualified Domain Name of the Domain Controller to take a transfer role to
 
 .PARAMETER Role
     Name of the role to transfer to domain controller
 
 .PARAMETER Transfer
     Transfers the specified role and give to specified domain controller. 
 
 .PARAMETER Seize
     Seize the specified role and give to specified domain controller.   
 
 .PARAMETER PassThru
     Show the FSMO role owners after performing action    
 
 .NOTES  
     Name: Set-FSMORoleOwner
     Author: Boe Prox
     DateCreated: 06/9/2011  
 
 .EXAMPLE
     Set-FSMORoleOwner -DomainController DC1.Rivendell.com -Role RidRole
     
     Description
     -----------
     Transfers the RidRole to DC1.Rivendell.com 
 
 .EXAMPLE
     Set-FSMORoleOwner -DomainController DC1.Rivendell.com -Role PdcRole -Transfer -PassThru
     
     NamingRole  : dc2.rivendell.com 
     Domain              : rivendell.com 
     RidRole            : dc2.rivendell.com 
     Forest              : rivendell.com 
     InfrastructureRole : dc2.rivendell.com 
     SchemaRole        : dc2.rivendell.com 
     PdcRole            : dc1.rivendell.com     
     
     Description
     -----------
     Transfers the PdcRole to DC1.Rivendell.com and displays the current FSMO Role Owners.
 
 .EXAMPLE
     Set-FSMORoleOwner -DomainController DC1.Rivendell.com -Role PdcRole,RidRole,SchemaRole -Transfer -PassThru
     
     NamingRole         : dc2.rivendell.com 
     Domain              : rivendell.com 
     RidRole            : dc1.rivendell.com 
     Forest              : rivendell.com 
     InfrastructureRole : dc2.rivendell.com 
     SchemaRole        : dc1.rivendell.com 
     PdcRole            : dc1.rivendell.com     
     
     Description
     -----------
     Transfers the PdcRole,RidRole and SchemaRole to DC1.Rivendell.com and displays the current FSMO Role Owners.  
     
 .EXAMPLE
     Set-FSMORoleOwner -DomainController DC1.Rivendell.com -Role PdcRole -Seize -PassThru
     
     WARNING: Performing this action is irreversible!
     The Domain Controller that originally holds this role should be rebuilt to avoid issues on the domain!
     
     NamingRole  : dc2.rivendell.com 
     Domain              : rivendell.com 
     RidRole            : dc2.rivendell.com 
     Forest              : rivendell.com 
     InfrastructureRole : dc2.rivendell.com 
     SchemaRole        : dc2.rivendell.com 
     PdcRole            : dc1.rivendell.com     
     
     Description
     -----------
     Seizes the PdcRole and places it on DC1.Rivendell.com and displays the current FSMO Role Owners.  
           
 #>
 [cmdletbinding(
     SupportsShouldProcess = 'True',
     ConfirmImpact = 'High',
     DefaultParameterSetName = 'Transfer'
     )] 
 Param (
     [parameter(Position=1,Mandatory = 'True',ValueFromPipeline = 'True',
         HelpMessage='Enter the Fully Qualified Domain Name of the Domain Controller')]
     [ValidateCount(1,1)]
     [string[]]$DomainController,
     [parameter(Position=2,Mandatory = 'True',
         HelpMessage = "InfrastructureRole,NamingRole,PdcRole,RidRole,SchemaRole")]
     [Alias('fsmo','fsmorole')]
     [ValidateSet('InfrastructureRole','NamingRole','PdcRole','RidRole','SchemaRole')]
     [ValidateCount(1,5)]
     [string[]]$Role,
     [parameter(Position=4,ParameterSetName='Transfer')]
     [Switch]$Transfer,    
     [parameter(Position=4,ParameterSetName='Seize')]
     [Switch]$Seize,
     [parameter(Position=5)]
     [switch]$PassThru
     )
 Begin {}
 Process {
     Try {
         Write-Verbose "Connecting to Forest"
         $forest = [system.directoryservices.activedirectory.Forest]::GetCurrentForest()
         Write-Verbose "Locating $DomainController" 
         $dc = $forest.domains | ForEach {
             $_.Domaincontrollers | Where {
                 $_.Name -eq $DomainController
                 }
             }
         }
     Catch {
         Write-Warning "$($Error)"
         Break
         }
     If (-NOT [string]::IsNullOrEmpty($dc)) {
         ForEach ($r in $role) {
             Switch ($PScmdlet.ParameterSetName) {
                "Transfer" {
                 Write-Verbose "Beginning transfer of $r to $DomainController"
                     If ($PScmdlet.ShouldProcess("$DomainController","Transfer Role: $($Role)")) {
                         Try {
                             $dc.TransferRoleOwnership($r)
                             }
                         Catch {
                             Write-Warning "$($Error[0])"
                             Break
                             }
                         }
                     }
                 "Seize" {
                     Write-Warning "Performing this action is irreversible!`nThe Domain Controller that originally holds this role should be rebuilt to avoid issues on the domain!"
                     Write-Verbose "Seizing $r and placing on $DomainController"
                     If ($PScmdlet.ShouldProcess("$DomainController","Seize Role: $($Role)")) {
                         Try {
                             $dc.SeizeRoleOwnership($r)
                             }
                         Catch {
                             Write-Warning "$($Error[0])"
                             Break
                             }
                         }               
                     }
                 Default {
                     Write-Warning "You must specify either -Transfer or -Seize!"
                     Break
                     }
                 }
             }
         }
     Else {
         Write-Warning "Unable to locate $DomainController!"
         Break
         }
     }
 End {
     If ($PSBoundParameters['PassThru']) {
         $forest = [system.directoryservices.activedirectory.Forest]::GetCurrentForest()
         ForEach ($domain in $forest.domains) {
             $forestproperties = @{
                 Forest = $Forest.name 
                 Domain = $domain.name 
                 SchemaRole = $forest.SchemaRoleOwner 
                 NamingRole = $forest.NamingRoleOwner 
                 RidRole = $Domain.RidRoleOwner 
                 PdcRole = $Domain.PdcRoleOwner 
                 InfrastructureRole = $Domain.InfrastructureRoleOwner 
                 }
             $newobject = New-Object PSObject -Property $forestproperties
             $newobject.PSTypeNames.Insert(0,"ForestRoles")
             $newobject        
             }
         }
     }
 }
`

