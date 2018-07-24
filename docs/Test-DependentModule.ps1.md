---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 3.0
Encoding: utf-8
License: cc0
PoshCode ID: 6360
Published Date: 
Archived Date: 2016-09-30t14
---

#  - 

## Description

ps module for creating new hires with functions for exchange online mailbox, skype voice provisioning, and um mailbox provisioning

## Comments



## Usage



## TODO



## module

`test-dependentmodule`

## Code

`#
 #
 <#
  .SYNOPSIS
     This module automates many of the tasks involved in processing a new starter on key Contoso systems.
  .DESCRIPTION 
     This module contains a number of helper and core functions for the purpose of provisioning a new starter on crucial Contoso systems like Active Directory, Office 365 and Lync/Skype.  Two main functions
     allow the user to either process a single new starter or import a group of new starters from a CSV file.  Please note that this module checks for and will warn the user if a number of pre-requisite modules
     are not present on the system.  The module will prompt for on-premise and Office 365 credentials if not found, and depends on an external script called Get-CSPhoneList.ps1 to provision a Skype phone number.
  .NOTES 
     Author  : TheDudeAbides
     Requires: PowerShell Version 3.0, Active Directory Module, Service Manager SMLets module, MSOnline module, Get-CsPhoneList.ps1 script for phone numbers
 #>
 
 $GLOBAL:smdefaultcomputer = "scsm.Contoso.com"
 $GLOBAL:SAMAccountRefreshTimestamp = (Get-Date).AddDays(-1)
 $GLOBAL:UPNRefreshTimeStamp = (Get-Date).AddDays(-1)
 $GLOBAL:AllSAMAccountNames = @()
 $GLOBAL:AllUPNs = @()
 $CompanyDomains = @("Contoso.com","TailSpinToys.com")
 $RegionMappings = @{'US'='AMER';'AR'='AMER';'PE'='AMER';'PA'='AMER';'CL'='AMER';'CA'='AMER';'AU'='ASIA';'NZ'='ASIA';'CN'='ASIA';'IN'='ASIA';'BH'='EMEA';'BE'='EMEA'`
                     ;'CY'='EMEA';'EG'='EMEA';'ET'='EMEA';'FJ'='ASIA';'IL'='EMEA';'IT'='EMEA';'JM'='AMER';'KW'='EMEA';'LS'='EMEA';'BN'='ASIA';'NP'='ASIA';'NL'='EMEA'`
                     ;'PK'='ASIA';'QA'='EMEA';'WS'='ASIA';'SG'='ASIA';'LK'='ASIA';'TW'='ASIA';'TR'='EMEA';'AE'='EMEA';'UK'='EMEA'}
 
 Function Test-DependentModule {
     If (Get-Module -ListAvailable -Name MSOnline) {
         Write-Host "MSOnline Module exists"
     } else {
     }
     If (Get-Module -ListAvailable -Name ActiveDirectory) {
         Write-Host "ActiveDirectory Module exists"
     } else {
         Write-Host "ActiveDirectory Module does not exist.  Please install it from https://www.microsoft.com/en-us/download/details.aspx?id=7887 before using this module." -ForegroundColor Red
     }
     If (Get-Module -ListAvailable -Name SMLets) {
         Write-Host "SMLets Module exists"
     } else {
         Write-Host "SMLets Module does not exist.  Please install System Center Service Manager Console before using this module." -ForegroundColor Red
     }
 }
 Test-DependentModule
 
 Function Connect-Office365 {
     $MSOLServiceCred = Get-MSOLCredential
 	$Session = New-PSSession -ConfigurationName microsoft.exchange -ConnectionUri https://outlook.office365.com/powershell-liveid/ -Credential $MSOLServiceCred -Authentication Basic -AllowRedirection
 	Import-PSSession $Session -AllowClobber -DisableNameChecking
 }
 
 Function Connect-Exchange2013 {
 Param(
     [Parameter(Position=0,Mandatory=$False)]
     [string]$ExchangeServer = "ExchangeServer.Contoso.com"
 )
 	$ExSession = New-PSSession -ConfigurationName Microsoft.Exchange -ConnectionUri "http://$ExchangeServer/PowerShell/" -Authentication Kerberos
 	Import-PSSession $ExSession -DisableNameChecking -AllowClobber
 }
 
 Function Connect-AzureAD {
 	$MSOLServiceCred = Get-MSOLCredential
 	Connect-MsolService -Credential $MSOLServiceCred
 }
 
 Function Connect-Skype {
     $DownLevelCred = Get-DownLevelCredential
     $SkypeSessionOption = New-PSSessionOption -SkipCACheck -SkipCNCheck -SkipRevocationCheck
     $SkypeSession = New-PSSession -ConnectionUri https://lyncserver.Contoso.com/ocspowershell -SessionOption $SkypeSessionOption -Credential $DownLevelCred -Name "SkypeServerSession"
     Import-PSSession $SkypeSession -DisableNameChecking -AllowClobber
 }
 
 Function Connect-DirSync {
 Param(
     [Parameter(Position=0,Mandatory=$False)]
     [string]$DirSyncServer = "Dirsync.Contoso.com"
 )
 	$DirSyncSession = New-PSSession -ComputerName $DirSyncServer -Authentication Kerberos
 	Import-PSSession $DirSyncSession -AllowClobber -DisableNameChecking
 }
 
 Function Test-Credential {
     <#
     .SYNOPSIS
         Takes a PSCredential object and validates it against the domain.
     .PARAMETER Credential
         A PScredential object with the username/password you wish to test. Typically this is generated using the Get-Credential cmdlet. Accepts pipeline input.
     .OUTPUTS
         A boolean, indicating whether the credentials were successfully validated.
     #>
     param(
         [parameter(Mandatory=$true,ValueFromPipeline=$true)]
         [System.Management.Automation.PSCredential]$credential
     )
     $Username = $credential.username
     $Password = $credential.GetNetworkCredential().password
  
     $CurrentDomain = "LDAP://" + ([ADSI]"").distinguishedName
     $Domain = New-Object System.DirectoryServices.DirectoryEntry($CurrentDomain,$Username,$Password)
     $DomainName = $Domain.name
  
     If ($DomainName -eq $null) {
     $false
     } else {
     $true
     }
     $Password = $null
 }
 
 Function Get-MSOLCredential {
  .SYNOPSIS 
     Checks for and imports a local encrypted credential file.
  .DESCRIPTION 
     This function checks for an encrypted XML file in the local directory to use for Cloud credentials.  If the file isn't found, the credentials are prompted for and then stored in the
     encrypted XML file.
  .NOTES 
     Author  : TheDudeAbides
     Requires: PowerShell Version 3.0, Active Directory Module
  .EXAMPLE
     Get-MSOLCredential
     #>
     If (Test-Path -Path ".\MSOLServiceCredential.xml") {
         $MSOLServiceCred = Import-Clixml .\MSOLServiceCredential.xml
         If (!(Test-Credential $MSOLServiceCred)) {
             Get-Credential -Message "Please enter a valid Office 365 username and password" | Export-Clixml .\MSOLServiceCredential.xml
             $MSOLServiceCred = Import-Clixml .\MSOLServiceCredential.xml
         }
     }
     Else {
         Get-Credential -Message "Please enter a valid Office 365 username and password" | Export-Clixml .\MSOLServiceCredential.xml
         $MSOLServiceCred = Import-Clixml .\MSOLServiceCredential.xml
     }
     Return $MSOLServiceCred
 }
 
 Function Get-DownLevelCredential {
  .SYNOPSIS 
     Checks for and imports a local encrypted credential file.
  .DESCRIPTION 
     This function checks for an encrypted XML file in the local directory to use for Domain credentials.  If the file isn't found, the credentials are prompted for and then stored in the
     encrypted XML file.
  .NOTES 
     Author  : TheDudeAbides
     Requires: PowerShell Version 3.0, Active Directory Module
  .EXAMPLE
     Get-DownLevelCredential
     #>
     If (Test-Path -Path ".\DownlevelCredential.xml") {
         $DownLevelCred = Import-Clixml .\DownlevelCredential.xml
         If (!(Test-Credential $DownLevelCred)) {
             Get-Credential -Message "Please enter credentials using the DOMAIN\username format" | Export-Clixml .\DownlevelCredential.xml
             $DownLevelCred = Import-Clixml .\DownlevelCredential.xml
         }
     }
     Else {
         Get-Credential -Message "Please enter credentials using the DOMAIN\username format" | Export-Clixml .\DownlevelCredential.xml
         $DownLevelCred = Import-Clixml .\DownlevelCredential.xml
     }
     Return $DownLevelCred
 }
 
 Function Get-RandomPassword {
  .SYNOPSIS 
     Generates a random password based on approved characters and a given length.
  .DESCRIPTION 
     This function generates a random password string that meets company requirements for length and complexity based on an approved character set.  Longer or shorter passwords can be generated
     based on an optional length however the function will throw an error if the requested password length is too short.
  .NOTES 
     Author  : TheDudeAbides
     Requires: PowerShell Version 3.0, Active Directory Module
  .EXAMPLE
     Get-RandomPassword -length 14
  .PARAMETER length
     Optional parameter for the length of the password to return.  If not specified the function defaults to a 10 character password.
     #>
 Param(
 [Parameter(Position=0,Mandatory=$False)]
 [int]$length=10
 )
 if ($length -lt 8) {
     throw "Password length is too short"
 }
 
 $numbers=$null
 For ($a=48;$a �le 57;$a++) {$numbers+=,[char][byte]$a }
 
 $uppercase=$null
 For ($a=65;$a �le 90;$a++) {$uppercase+=,[char][byte]$a }
 
 $lowercase=$null
 For ($a=97;$a �le 122;$a++) {$lowercase+=,[char][byte]$a }
 
 $specialchars='!#%$&~?<>+-:;'.ToCharArray()
 
 $Buffer = @()
 For ($a=1;$a �le $length;$a++) {$Buffer+=0 }
 
 while ($true) {
 $CharNum = (Get-Random -minimum 0 -maximum $length)
 if ($Buffer[$CharNum] -eq 0) {$Buffer[$CharNum] = 1; break}
 }
 
 while ($true) {
 $CharNum = (Get-Random -minimum 0 -maximum $length)
 if ($Buffer[$CharNum] -eq 0) {$Buffer[$CharNum] = 2; break}
 }
 
 while ($true) {
 $CharNum = (Get-Random -minimum 0 -maximum $length)
 if ($Buffer[$CharNum] -eq 0) {$Buffer[$CharNum] = 3; break}
 }
 
 while ($true) {
 $CharNum = (Get-Random -minimum 0 -maximum $length)
 if ($Buffer[$CharNum] -eq 0) {$Buffer[$CharNum] = 4; break}
 }
 
 $Password = ""
 foreach ($CharType in $Buffer) {
     if ($CharType -eq 0) {$CharType = ((1,2,3,4)|Get-Random)}
         switch ($CharType) {
         1 {$Password+=($numbers | Get-Random)}
         2 {$Password+=($uppercase | Get-Random)}
         3 {$Password+=($lowercase | Get-Random)}
         4 {$Password+=($specialchars | Get-Random)}
         }
     }
     return $Password
 }
 
 Function Get-AllSAMAccountNames {
  .SYNOPSIS 
     Queries Active Directory for all SAMAccountNames.
  .DESCRIPTION 
     This function queries Active Directory for all SAMAccountNames and stores them in a variable for use by other functions.
  .NOTES 
     Author  : TheDudeAbides
     Requires: PowerShell Version 3.0, Active Directory Module
  .EXAMPLE
     Get-AllUPNs
     #>
     $GLOBAL:SAMAccountRefreshTimestamp = Get-Date
     Write-Verbose "Refreshing SAM Accounts..."
     $GLOBAL:AllSAMAccountNames = (Get-ADUser -Filter * | Select-Object SAMAccountName).SAMAccountName
     Write-Verbose "Refresh complete."    
 }
 
 Function Get-AllUPNs {
  .SYNOPSIS 
     Queries Active Directory for all UserPrincipalNames.
  .DESCRIPTION 
     This function queries Active Directory for all UserPrincipalNames and stores them in a variable for use by other functions.
  .NOTES 
     Author  : TheDudeAbides
     Requires: PowerShell Version 3.0, Active Directory Module
  .EXAMPLE
     Get-AllUPNs
     #>
     $GLOBAL:UPNRefreshTimeStamp = Get-Date
     Write-Verbose "Refreshing UPNs..."
     $GLOBAL:AllUPNs = (Get-ADUser -Filter * | Select-Object UserPrincipalName).UserPrincipalName
     Write-Verbose "Refresh complete."    
 }
 
 Function Get-UniqueSAMAccountName {
  .SYNOPSIS 
     Queries Active Directory for a unique SAMAccountName based on provided attributes.
  .DESCRIPTION 
     This function queries Active Directory for a unique SAMAccountName based on provided information.  The function checks to see if it's local cache of SAMAccountNames is older than 5 minutes and will
     update if necessary.  Then based on a recent cache of SAMAccountNames, the function will calculate several possible SAMAccountName combinations and compare the results against the cache.  If a unique
     value is found the function will return the value.  If a unique value could not be determined, the function will throw an error.
  .NOTES 
     Author  : TheDudeAbides
     Requires: PowerShell Version 3.0, Active Directory Module
  .EXAMPLE
     Get-UniqueSAMAccountName -GivenName John -Surname Smith
  .EXAMPLE
     Get-UniqueSAMAccountName -GivenName John -Initial A -Surname Smith
  .PARAMETER GivenName
     Required parameter for the first name of the user. Apostrophes should be ommitted from the name to ensure compatibility with IT systems. 
  .PARAMETER Surname
     Required parameter for the last name of the user. Apostrophes should be ommitted from the name to ensure compatibility with IT systems.
  .PARAMETER Initial
     Optional parameter for the first letter of the user's middle initial.
     #>
     Param(
     [Parameter(Position=0,Mandatory=$True,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
     [string]$GivenName,
     [Parameter(Position=1,Mandatory=$True,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
     [string]$Surname,
     [Parameter(Position=2,Mandatory=$False,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
     [string]$Initial
     )
 
     If ((Get-Date) -gt ($GLOBAL:SAMAccountRefreshTimestamp.AddMinutes(5))) {
         Get-AllSAMAccountNames
     }
 
     If ($GLOBAL:AllSAMAccountNames.Length -lt 10) {
         throw "SAM Account List is blank, unable to refresh from Active Directory"
     }
 
     $GivenName = $GivenName.Replace("'","")
     $GivenName = $GivenName.Split(" ")[0]
     $Surname = $Surname.Replace("'","")
     $Surname = $Surname.Replace(" ","")
     
     $ComboArray = @()
     $ComboArray += ,@(6,1,'')
     $ComboArray += ,@(6,2,'')
     $ComboArray += ,@(5,2,'')
     $ComboArray += ,@(5,3,'')
     $ComboArray += ,@(6,3,'')
     $ComboArray += ,@(5,4,'')
     $ComboArray += ,@(4,4,'')
     $ComboArray += ,@(6,2,'x')
     $ComboArray += ,@(5,3,'x')
 
     $ValidSAMAccountName = $false
     If ($Initial -match "\w+") {
         Foreach ($Combo in $ComboArray) {
             $PotentialSAMAccountName = ([regex]"\w{1,$($combo[0])}").Match($Surname).Groups[0].Value + ([regex]"\w{1,$($combo[1])}").Match($GivenName).Groups[0].Value + $Initial.Substring(0,1)
             $result = $GLOBAL:AllSAMAccountNames | Where-Object {$_ -eq $PotentialSAMAccountName}
             If ($result -eq $null) {
                 $ValidSAMAccountName = $true
                 $PotentialSAMAccountName = $PotentialSAMAccountName.ToLower()
                 break
             }
         }
     }
     Else {
         Foreach ($Combo in $ComboArray) {
             $PotentialSAMAccountName = ([regex]"\w{1,$($combo[0])}").Match($Surname).Groups[0].Value + ([regex]"\w{1,$($combo[1])}").Match($GivenName).Groups[0].Value
             $result = $GLOBAL:AllSAMAccountNames | Where-Object {$_ -eq $PotentialSAMAccountName}
             If ($result -eq $null) {
                 $ValidSAMAccountName = $true
                 $PotentialSAMAccountName = $PotentialSAMAccountName.ToLower()
                 break
             }
         }
     }
 
     If (($ValidSAMAccountName -eq $true) -and ($PotentialSAMAccountName -match "^\w{2,9}$")) {
         return $PotentialSAMAccountName
     }
     Else {
         throw "No valid SAMAccountName was able to be generated" 
     }
 }
 
 Function Get-UniqueUPN {
  .SYNOPSIS 
     Queries Active Directory for a unique UserPrincipalName based on provided attributes.
  .DESCRIPTION 
     This function queries Active Directory for a unique userprincipalname based on provided information.  The function checks to see if it's local cache of UPNs is older than 5 minutes and will
     update if necessary.  Then based on a recent cache of UPNs, the function will calculate several possible UserPrincipalName combinations and compare the results against the cache.  If a unique
     value is found the function will return the value.  If a unique value could not be determined, the function will throw an error.
  .NOTES 
     Author  : TheDudeAbides
     Requires: PowerShell Version 3.0, Active Directory Module
  .EXAMPLE
     Get-UniqueUPN -GivenName John -Surname Smith -Company Contoso
  .EXAMPLE
     Get-UniqueUPN -GivenName John -Initial A -Surname Smith -Company TailSpinToys
  .PARAMETER GivenName
     Required parameter for the first name of the user. Apostrophes should be ommitted from the name to ensure compatibility with IT systems. 
  .PARAMETER Surname
     Required parameter for the last name of the user. Apostrophes should be ommitted from the name to ensure compatibility with IT systems.
  .PARAMETER Initial
     Optional parameter for the first letter of the user's middle initial.
  .PARAMETER Company
     Required parameter for the sub-company of the user. This will be used to populate the UPN suffix; valid entries are 'Contoso','TailSpinToys','Hawksley','Burton','Slayden','JV'.
 #>
     Param(
     [Parameter(Position=0,Mandatory=$True,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
     [string]$GivenName,
     [Parameter(Position=1,Mandatory=$True,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
     [string]$Surname,
     [Parameter(Position=2,Mandatory=$False,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
     [string]$Initial,
     [Parameter(Position=3,Mandatory=$True,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)][ValidateSet('Contoso','TailSpinToys','JV')]
     [string]$Company
     )
 
     If ((Get-Date) -gt ($GLOBAL:UPNRefreshTimeStamp.AddMinutes(5))) {
         Get-AllUPNs
     }
 
     If ($GLOBAL:AllUPNs.Length -lt 10) {
         throw "UPN List is blank, unable to refresh from Active Directory"
     }
 
     $GivenName = $GivenName.Replace("'","")
     $GivenName = $GivenName.Split(" ")[0]
     $Surname = $Surname.Replace("'","")
     $Surname = $Surname.Replace(" ","")
 
     If ($CompanyDomains -match $Company) {
         $DomainName = ($CompanyDomains -match $Company)
     }
     Else {
         $DomainName = "Contoso.com"
     }
 
     If ($Initial -match "\w+") {
         $PotentialUPN = $GivenName + "." + $Initial + "." + $Surname + "@" + $DomainName
     }
     Else {
         $PotentialUPN = $GivenName + "." + $Surname + "@" + $DomainName
     }
     $result = $GLOBAL:AllUPNs | Where-Object {$_ -eq $PotentialUPN}
     If ($result -eq $null) {
         return $PotentialUPN
     } 
     Else {
         $PotentialUPN = $GivenName + "." + $Surname + "@" + $DomainName
         $result = $GLOBAL:AllUPNs | Where-Object {$_ -eq $PotentialUPN}
         If ($result -eq $null) {
             return $PotentialUPN
         } 
         Else {
             $PotentialUPN = $GivenName + "." + "X" + "." + $Surname + "@" + $DomainName
             $result = $GLOBAL:AllUPNs | Where-Object {$_ -eq $PotentialUPN}
             If ($result -eq $null) {
                 return $PotentialUPN
             }
             Else {
                 throw "No valid UPN was able to be generated"
             }
         }
     }
 }
 
 Function Invoke-DirSync {
  .SYNOPSIS 
     Runs a manual sync for the Azure AD Connect server when called.
  .DESCRIPTION 
     This function uses PowerShell Remoting to start a manual delta sync on the Azure AD Connect server.  The server can be specified or the default accepted for the Contoso domain.  If
     running the function against an Azure AD Connect Server in another domain, the user running the cmdlet should have rights to remote into and execute AAD Connect cmdlets on that server.
  .NOTES 
     Author  : TheDudeAbides
     Requires: PowerShell Version 3.0
  .EXAMPLE
     Invoke-DirSync
  .EXAMPLE
     Invoke-DirSync -DirSyncServer "12345-Dirsync01.contoso.com"
  .PARAMETER DirSyncServer
     Optional parameter for when using an alternate Azure AD Connect server.  A full FQDN is recommended to ensure the correct server can be found.
     #>
 Param(
 [Parameter(Position=0,Mandatory=$False)]
 [string]$DirSyncServer = "dirsync.Contoso.com",
 [Parameter(Position=1,Mandatory=$False)]
 [int]$TimetoWait = 60
 )
     For ($TimetoWait=60; $TimetoWait -gt 1; $TimetoWait--) {
       Write-Progress -Activity "Waiting $TimetoWait seconds for Active Directory synchronization..." `
        -SecondsRemaining $TimetoWait -Status "Please wait."
       Start-Sleep 1
     }
 
     $DirSyncSession = New-PSSession -ComputerName $DirSyncServer -Authentication Kerberos
     Invoke-Command -Session $DirSyncSession -ScriptBlock {Import-Module -Name adsync;$ErrorActionPreference = 'SilentlyContinue'}
     Invoke-Command -Session $DirSyncSession -ScriptBlock {Start-ADSyncSyncCycle -PolicyType Delta;$ErrorActionPreference = 'Continue'}
     
     For ($TimetoWait=60; $TimetoWait -gt 1; $TimetoWait--) {
       Write-Progress -Activity "Waiting $TimetoWait seconds for DirSync to complete..." `
        -SecondsRemaining $TimetoWait -Status "Please wait."
       Start-Sleep 1
     }
 }
 
 Function Confirm-ValidUser {
  .SYNOPSIS 
     Checks user input for valid data.
  .DESCRIPTION 
     This function cleans up and checks incoming data for a new user for valid input so that it can be passed to other functions.  If any unrecoverable errors are found, the function will
     throw an appropriate error.  The object returned should match all fields that were given to it.
  .NOTES 
     Author  : TheDudeAbides
     Requires: PowerShell Version 3.0
  .EXAMPLE
     Confirm-ValidUser -GivenName John -Surname Smith -EmployeeID 12345 -EmployeeType F -OfficeCode "US-DEN-1"
  .EXAMPLE
     Confirm-ValidUser -GivenName John -Surname Smith -Initial A -EmployeeID 12345 -EmployeeType F -BU 11111 -OfficeCode "US-DEN-1"
  .PARAMETER GivenName
     Required parameter for the first name of the user. Apostrophes should be ommitted from the name to ensure compatibility with IT systems.
  .PARAMETER Surname
     Required parameter for the last name of the user. Apostrophes should be ommitted from the name to ensure compatibility with IT systems.
  .PARAMETER Initial
     Optional parameter for the first letter of the user's middle name.
  .PARAMETER PreferredGivenName
     Optional parameter for a preferred first name provided by the user upon hire.  This will only populate the Display Name, and does not change the logon attributes for the user.
  .PARAMETER EmployeeID
     Required parameter for the new user's employee number.  Valid entries should be either between 5-8 digits, a C for contractor or a dash if unknown.
  .PARAMETER EmployeeType
     Required parameter for the employee type.  Valid examples include F for Full-time, T for Temporary, C for Contractor, JV for a Joint Venture user, or - if unknown.
  .PARAMETER Description
     Optional parameter for the user description.  Often includes the title and office code for the user, but can be any additional information such as the requestor of a contractor account, etc.
  .PARAMETER BU
     Optional parameter for the Business Unit of the new user.  Valid entries are a 5 digit number only, and if not provided will usually be filled in by sync from EID.
  .PARAMETER OfficeCode
     Required parameter for the office code of the new user.  Valid entries should start with a two letter country code and then a dash.  For most new users, this is later replaced by
     the sync from EID, but should be provided so that the Office 365 country code can be filled in appropriately.
  .PARAMETER Company
     Required parameter for the sub-company of the new user.  Valid Entries include Contoso, TailSpinToys, Hawksley, JV.  This is required to set the primary email address, Skype
     address and UPN correctly.
  .PARAMETER SRNumber
     Optional parameter for the Service Request number of the new user.  If provided PowerShell will attempt to update the Service Request ticket with the information from the new starter script.
  .PARAMETER EnableMailbox
     Optional parameter indicating if the user should be enabled for a mailbox or not.  A value of 'N' or 'No' will cause the functions to not create the mailbox.  Anything else entered or nothing at all will
     cause the script to assume an account is to be created.
  .PARAMETER EnableSkype
     Optional parameter indicating if the user should be enabled for a skype account or not.  A value of 'N' or 'No' will cause the functions to not create the skype account.  Anything else entered or nothing at all will
     cause the script to assume an account is to be created.
     #>
     [cmdletbinding()]
 Param(
         [Parameter(Position=0,Mandatory=$True,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
         [string]$GivenName="",
         [Parameter(Position=1,Mandatory=$True,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
         [string]$Surname="",
         [Parameter(Position=2,Mandatory=$False,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
         [string]$Initial="",
         [Parameter(Position=3,Mandatory=$False,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
         [string]$PreferredGivenName="",
         [Parameter(Position=4,Mandatory=$True,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
         [string]$EmployeeID="",
         [Parameter(Position=5,Mandatory=$True,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)][ValidateSet('F','T','C','JV','-')]
         [string]$EmployeeType="",
         [Parameter(Position=6,Mandatory=$False,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
         [string]$Description="",
         [Parameter(Position=7,Mandatory=$False,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)][ValidateRange(00000,99999)]
         [int]$BU="",
         [Parameter(Position=8,Mandatory=$True,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
         [string]$OfficeCode="",
         [Parameter(Position=9,Mandatory=$False,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)][ValidateSet('Contoso','TailSpinToys','JV')]
         [string]$Company="Contoso",
         [Parameter(Position=10,Mandatory=$False,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
         [string]$SRNumber="",
         [Parameter(Position=11,Mandatory=$False,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
         [string]$EnableMailbox="",
         [Parameter(Position=12,Mandatory=$False,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
         [string]$EnableSkype=""
     )
     $UserObject = New-Object -TypeName psobject -Property @{
         EmployeeID = $EmployeeID.Trim()
         OfficeCode = $OfficeCode.Trim()
         BU = $BU
         Surname = $Surname.Trim()
         PreferredGivenName = $PreferredGivenName.Trim()
         GivenName = $GivenName.Trim()
         Initial = $Initial.Trim()
         Company = $Company.Trim()
         EmployeeType = $EmployeeType.Trim()
         Description = $Description.Trim()
         ValidInput = $False
         FullName = ""
         SRNumber = $SRNumber.Trim()
         EnableSkype = $EnableSkype
         EnableMailbox = $EnableMailbox
     }
 
     If ($Initial -match "\w+") {
         $FullName = $UserObject.GivenName + " " + $UserObject.Initial + " " + $UserObject.Surname
     } 
     Else {
         $FullName = $UserObject.GivenName + " " + $UserObject.Surname
     }
 
     If (!(($UserObject.EmployeeID -match "^\d{5,8}$") -or ($UserObject.EmployeeID -match "c") -or ($UserObject.EmployeeID -eq "-"))) {
         throw "$FullName with ID ($UserObject.EmployeeID) has invalid Employee ID, valid entries are 5-8 digits, 'c' or '-'"    
     } 
     Elseif (!(($UserObject.OfficeCode -match "^\w{2}\-\w{2}") -or ($UserObject.OfficeCode -eq "-"))) {
         throw "$FullName with ID ($UserObject.EmployeeID) has invalid Office Code"
     } 
     Else {
         $UserObject.FullName = $FullName
         $UserObject.ValidInput = $True
         return $UserObject
     }
 }
 
 Function Update-SCSMStarterTicket {
  .SYNOPSIS 
     Updates the SCSM Activities for a given new starter.
  .DESCRIPTION 
     This function updates the activities for a new starter if required.  The function requires the ticket number for the starter and key pieces of information in ordert to populate the
     notes on the activities.  All fields are required except SkyperNumber in order to ensure the ticket is updated correctly, and the SR itself will not be closed out until manual review is 
     performed.  Quotation marks are recommended around all fields to ensure that the data is read in correctly.
  .NOTES 
     Author  : TheDudeAbides
     Requires: PowerShell Version 3.0, SMLets module for Service Manager
  .EXAMPLE
     Update-SCSMStarterTicket -SRNumber SR12345 -UserPrincipalName "John.Tester@Contoso.com" -SAMAccountName testerj -SkypeNumber "+1 720 887 8700"
  .EXAMPLE
     Update-SCSMStarterTicket -SRNumber SR12345 -UserPrincipalName "John.Tester@Contoso.com" -SAMAccountName testerj
  .PARAMETER SRNumber
     Required parameter for the ticket in Service Manager to update.  If no ticket exists, one will need to be created before this can be run.
  .PARAMETER UserPrincipalName
     Required parameter for the UPN of the user.  This will be added to the AD/O365/Lync Activities linked in the ticket.  Should these differ, the ticket should be manually changed
     to reflect the discrepancy.
  .PARAMETER SAMAccountName
     Required parameter for the SAMAccountName of the user.  This will be added to the AD and E1 activities in the ticket and is required before an E1 login can be provisioned.
  .PARAMETER SkypeNumber
     Optional parameter for the Skype/Lync number assigned to the user.  This only needs to be populated if the Office location is Lync Enterprise Voice enabled.  This can be in any format required but should
     be enclosed in quotation marks to ensure that the data is input correctly.
     #>
 Param(
 [Parameter(Position=0,Mandatory=$True)]
 [string]$SRNumber,
 [Parameter(Position=1,Mandatory=$True)]
 [string]$UserPrincipalName,
 [Parameter(Position=2,Mandatory=$True)]
 [string]$SAMAccountName,
 [Parameter(Position=3,Mandatory=$False)]
 [string]$SkypeNumber
 )
     Write-Host "Updating Service Manager Activities in ticket $SRNumber for Employee: $UserPrincipalName" -ForegroundColor Green
     $serviceRequestClass = Get-SCSMClass -name System.WorkItem.ServiceRequest$
     $manualActivityClass = Get-SCSMClass -Name System.WorkItem.Activity.ManualActivity$
         
     $ServiceRequest = Get-SCSMObject -Class $serviceRequestClass -Filter "ID -eq $SRNumber"
     If ($ServiceRequest.DisplayName -match "Starter") {
         $ManualActivities = Get-SCSMRelationshipObject -BySource $serviceRequest | Select-Object sourceobject -unique | Where-Object {$_.SourceObject -like "MA*"}
         $ManualActivityIDs = $ManualActivities | ForEach-Object {$string=$_.sourceobject.tostring();$string.split(':')[0]}
 
         Foreach ($ActID in $ManualActivityIDs) {
             $MAObject = Get-SCSMObject -Class $manualActivityClass -Filter "ID -eq $ActID"
             Switch -Wildcard ($MAObject.Title) {
                 "*AD Activities*" {
                     Get-SCSMObject -Class $manualActivityClass -Filter "ID -eq $ActID" | Set-SCSMObject -PropertyHashtable @{'Status'='Completed';'Notes'="Activity Completed. `nUserPrincipalName set to $UserPrincipalName `nSAMAccountName set to $SAMAccountName"}
                     }
                 "*E1*" {
                     $MADescription = $MAObject.Description
                     Get-SCSMObject -Class $manualActivityClass -Filter "ID -eq $ActID" | Set-SCSMObject -PropertyHashtable @{'Status'='Active';'Description'="$MADescription `nSAMAccountName generated as $SAMAccountName"}
                     }
                 "*O365*" {
                     Get-SCSMObject -Class $manualActivityClass -Filter "ID -eq $ActID" | Set-SCSMObject -PropertyHashtable @{'Status'='Completed';'Notes'="Activity Completed. `nSIP address set to $UserPrincipalName"}
                     }
                 "*Enterprise Voice*" {
                     If (!($SkypeNumber -eq $null)) {
                         Get-SCSMObject -Class $manualActivityClass -Filter "ID -eq $ActID" | Set-SCSMObject -PropertyHashtable @{'Status'='Completed';'Notes'="Activity Completed. `nLineURI set to $SkypeNumber"}
                     }
                     Else {
                         Get-SCSMObject -Class $manualActivityClass -Filter "ID -eq $ActID" | Set-SCSMObject -Property Status -Value Active
                     }
                     }
                 "*Communication*" {
                     Get-SCSMObject -Class $manualActivityClass -Filter "ID -eq $ActID" | Set-SCSMObject -Property Status -Value Active
                     }
                 "*IFS Access*" {
                     Get-SCSMObject -Class $manualActivityClass -Filter "ID -eq $ActID" | Set-SCSMObject -Property Status -Value Active
                     }
             }
         }
     }
     Else {
         throw "Service Request $SRNumber could not be found"
     }
 }
 
 Function New-ContosoStarter {
  .SYNOPSIS 
     Creates the necessary accounts for a new user in Active Directory, Exchange/Office 365, and Skype for Business.
  .DESCRIPTION 
     This function takes in user input and kicks off the processes required to create a new user on several key Contoso systems.  The script will create an AD account with the various required
     parameters, enable a Hybrid mailbox for that person, provision the necessary Office 365 license, configure a Skype for Business account and enable a phone number/voicemail box if the office
     that the user is located in is enabled for Enterprise Voice.  Due to sync processes, please note that the function can take up to 10-15 minutes to run.
  .NOTES 
     Author  : TheDudeAbides
     Requires: PowerShell Version 3.0, Active Directory PowerShell module, Remote connections to Exchange/Lync, MSOnline Module for Office 365 provisioning, SMLets module for Service Manager
  .EXAMPLE
     New-ContosoStarter -GivenName John -Surname Smith -EmployeeID 12345 -EmployeeType F -OfficeCode "US-DEN-1"
  .EXAMPLE
     New-ContosoStarter -GivenName John -Surname Smith -Initial A -EmployeeID 12345 -EmployeeType F -BU 11111 -OfficeCode "US-DEN-1"
  .PARAMETER GivenName
     Required parameter for the first name of the user. Apostrophes should be ommitted from the name to ensure compatibility with IT systems.
  .PARAMETER Surname
     Required parameter for the last name of the user. Apostrophes should be ommitted from the name to ensure compatibility with IT systems.
  .PARAMETER Initial
     Optional parameter for the first letter of the user's middle name.
  .PARAMETER PreferredGivenName
     Optional parameter for a preferred first name provided by the user upon hire.  This will only populate the Display Name, and does not change the logon attributes for the user.
  .PARAMETER EmployeeID
     Required parameter for the new user's employee number.  Valid entries should be either between 5-8 digits, a C for contractor or a dash if unknown.
  .PARAMETER EmployeeType
     Required parameter for the employee type.  Valid examples include F for Full-time, T for Temporary, C for Contractor, JV for a Joint Venture user, or - if unknown.
  .PARAMETER Description
     Optional parameter for the user description.  Often includes the title and office code for the user, but can be any additional information such as the requestor of a contractor account, etc.
  .PARAMETER BU
     Optional parameter for the Business Unit of the new user.  Valid entries are a 5 digit number only, and if not provided will usually be filled in by sync from EID.
  .PARAMETER OfficeCode
     Required parameter for the office code of the new user.  Valid entries should start with a two letter country code and then a dash.  For most new users, this is later replaced by
     the sync from EID, but should be provided so that the Office 365 country code can be filled in appropriately.
  .PARAMETER Company
     Required parameter for the sub-company of the new user.  Valid Entries include Contoso, TailSpinToys, Hawksley, JV, Slayden, or Burton.  This is required to set the primary email address, Skype
     address and UPN correctly.
  .PARAMETER SRNumber
     Optional parameter for the Service Request number of the new user.  If provided PowerShell will attempt to update the Service Request ticket with the information from the new starter script.
     #>
     [cmdletbinding()]
     Param(
         [Parameter(Position=0,Mandatory=$True,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
         [string]$GivenName="",
         [Parameter(Position=1,Mandatory=$True,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
         [string]$Surname="",
         [Parameter(Position=2,Mandatory=$False,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
         [string]$Initial="",
         [Parameter(Position=3,Mandatory=$False,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
         [string]$PreferredGivenName="",
         [Parameter(Position=4,Mandatory=$False,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
         [string]$EmployeeID="",
         [Parameter(Position=5,Mandatory=$True,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)][ValidateSet('F','T','C','JV','-')]
         [string]$EmployeeType="",
         [Parameter(Position=6,Mandatory=$False,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
         [string]$Description="",
         [Parameter(Position=7,Mandatory=$False,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)][ValidateRange(00000,99999)]
         [int]$BU="",
         [Parameter(Position=8,Mandatory=$False,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
         [string]$OfficeCode="",
         [Parameter(Position=9,Mandatory=$False,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)][ValidateSet('Contoso','TailSpinToys','JV')]
         [string]$Company="Contoso",
         [Parameter(Position=10,Mandatory=$False,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
         [string]$EnableMailbox="",
         [Parameter(Position=11,Mandatory=$False,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
         [string]$EnableSkype="",
         [Parameter(Position=12,Mandatory=$False,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
         [string]$SRNumber        
     )
     $ExecutionStart = Get-Date
     Write-Output "New account creation started at $ExecutionStart"
     $StarterObject = New-Object -TypeName psobject -Property @{
         EmployeeID = "$EmployeeID"
         OfficeCode = "$OfficeCode"
         BU = $BU
         Surname = "$Surname"
         PreferredGivenName = "$PreferredGivenName"
         GivenName = "$GivenName"
         Initial = "$Initial"
         Company = "$Company"
         EmployeeType = "$EmployeeType"
         EnableSkype = $EnableSkype
         EnableMailbox = $EnableMailbox
         Description = "$Description"
         SRNumber = "$SRNumber"
     }
     Try {
         $UserResult = $StarterObject | Confirm-ValidUser
     }
     Catch {
         $ErrorMessage = $_.Exception.Message
         $FailedItem = $_.Exception.ItemName
         exit
     }
     
     Write-Output "Attempting to create AD account for $($UserResult.FullName)"
     $ADresult = $UserResult | New-ContosoADUser
     Write-Host "$($ADresult.UserPrincipalName) created successfully" -ForegroundColor Green
     
     If (!(($StarterObject.EnableSkype -eq 'N') -or ($StarterObject.EnableSkype -eq 'No'))) {
         Write-Output "Attempting to create Skype account for $($UserResult.FullName)"
         $SkypeResult = Enable-ContosoSkypeUser -UserPrincipalName $ADresult.UserPrincipalName
         If ($($SkypeResult.LineURI) -like "tel*") {
             $ADPhoneNumber = $SkypeResult.LineURI.Split(':')[1]
             $ADPhoneNumber = $ADPhoneNumber.Split(';')[0]
             Set-Aduser -Identity $SkypeResult.Identity -OfficePhone $ADPhoneNumber
         }
     }
    If (!(($StarterObject.EnableMailbox -eq 'N') -or ($StarterObject.EnableMailbox -eq 'No'))) {
         Write-Output "Attempting to create mailbox for $($UserResult.FullName)"
         Enable-ContosoMailbox -UserPrincipalName $ADresult.UserPrincipalName
         $Country = $ADresult.Office.SubString(0,2)
         Invoke-DirSync
         Enable-ContosoO365License -UserPrincipalName $ADresult.UserPrincipalName -Country $Country
     }
     
     Write-Host "Pausing for 30 seconds to allow Exchange Online to provision mailboxes" -ForegroundColor Cyan
     Start-Sleep -Seconds 30
 
     If ($($SkypeResult.LineURI) -like "tel*") {
         Write-Output "Attempting to create UM Mailbox for $($UserResult.FullName)"
         $MbxResult = Enable-ContosoUMMailbox -UserPrincipalName $ADresult.UserPrincipalName -Office $ADresult.Office
     }
 
     $FinishedAccount = New-Object -TypeName psobject -Property @{
         Path = $ADresult.Path
         DisplayName = $ADresult.DisplayName
         EmployeeID = "$EmployeeID"
         OfficeCode = "$OfficeCode"
         BU = $BU
         GivenName = $GivenName
         Surname = $Surname
         SamAccountName = $ADresult.SamAccountName
         UserPrincipalName = $ADresult.UserPrincipalName
         SkypeNumber = $ADPhoneNumber
         Company = "$Company"
         EmployeeType = "$EmployeeType"
         Description = "$Description"
         EnableSkype = $EnableSkype
         EnableMailbox = $EnableMailbox
         Password = $ADresult.AccountPassword
         SRNumber = $UserResult.SRNumber
     }
 
     If ($SRNumber -match "^SR*") {
         Update-SCSMStarterTicket -SRNumber $SRNumber -UserPrincipalName $FinishedAccount.UserPrincipalName -SAMAccountName $FinishedAccount.SamAccountName
     }
     Elseif (($SRNumber -match "^SR*") -and ($ADPhoneNumber -match "\d{4,}")) {
         Update-SCSMStarterTicket -SRNumber $SRNumber -UserPrincipalName $FinishedAccount.UserPrincipalName -SAMAccountName $FinishedAccount.SamAccountName -SkypeNumber $ADPhoneNumber
     }
 
     $defaultProperties = @('DisplayName','EmployeeID','SamAccountName','UPN','SkypeNumber','Password')
     $defaultDisplayPropertySet = New-Object System.Management.Automation.PSPropertySet(�DefaultDisplayPropertySet�,[string[]]$defaultProperties)
     $PSStandardMembers = [System.Management.Automation.PSMemberInfo[]]@($defaultDisplayPropertySet)
     $FinishedAccount | Add-Member MemberSet PSStandardMembers $PSStandardMembers
     
     $ExecutionStop = Get-Date
     $Timetaken = $ExecutionStop - $ExecutionStart
     $TimetakenInMin = "{0:N0}" -f ($Timetaken.TotalMinutes)
     Write-Output "Script completed in $TimetakenInMin minutes`n"
     Get-PSSession | Remove-PSSession -Confirm:$False
     return $FinishedAccount
 }
 
 Function Import-ContosoStarterFile {
  .SYNOPSIS 
     Creates the necessary accounts for a new user in Active Directory, Exchange/Office 365, and Skype for Business based on a CSV import
  .DESCRIPTION 
     This function takes in user accounts from a CSV file and creates new AD, Office 365 and Skype for Business accounts based on the parameters for each user in the file.  .
  .NOTES 
     Author  : TheDudeAbides
     Requires: PowerShell Version 3.0, Active Directory PowerShell module, Remote connections to Exchange/Lync, MSOnline Module for Office 365 provisioning
  .EXAMPLE
     Import-ContosoStarterFile -CSVFilePath .\NewStarters.csv
  .PARAMETER CSVFilePath
     Required parameter for the path to the CSV file that contains the new user details.  The file should contain the following Column headers, the order is not important: EmployeeID, Officecode
     BU, Surname, GivenName, PreferredGivenName, Initial, Company, EmployeeType, Description.  At a minimum GivenName, Surname and EmployeeType are required and all other fields can be different
     depending on new starter information provided.
     #>
     Param(
         [Parameter(Position=0,Mandatory=$True,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
         [string]$CSVFilePath=""
     )
     $ExecutionStart = Get-Date
     Write-Output "New account creation started at $ExecutionStart"
     $CSVImportArray = @()
     $ValidUserArray = @()
     $ADResultArray = @()
     $SkypeResultArray = @()
     try {
         $CSVImportArray = Import-Csv $CSVFilePath
     }
     catch {
         Write-Output "$($_.Exception.Message)"
         exit
     }
 
     Foreach ($UserObject in $CSVImportArray) {
         Try {
             $UserResult = $UserObject | Confirm-ValidUser 
             $ValidUserArray = $ValidUserArray + $UserResult
         }
         Catch {
             Write-Output "$($_.Exception.Message)"
         }
     }
         
     Foreach ($Line in $ValidUserArray) {
         Write-Output "Attempting to create AD account for $($Line.FullName)"
         $ADresult = $Line | New-ContosoADUser
         Write-Host "$($ADresult.UserPrincipalName) created successfully" -ForegroundColor Green
         $FinishedAccount = New-Object -TypeName psobject -Property @{
             Path = $ADresult.Path
             DisplayName = $ADresult.DisplayName
             GivenName = $line.GivenName
             Surname = $line.Surname
             EmployeeID = $line.EmployeeID
             OfficeCode = $line.OfficeCode
             BU = $line.BU
             SamAccountName = $ADresult.SamAccountName
             UserPrincipalName = $ADresult.UserPrincipalName
             SkypeNumber = $null
             Company = $line.Company
             EnableSkype = $line.EnableSkype
             EnableMailbox = $line.EnableMailbox
             SRNumber = $line.SRNumber
             EmployeeType = $line.EmployeeType
             Description = $line.Description
             Password = $ADresult.AccountPassword
         }
         $ADResultArray = $ADResultArray + $FinishedAccount
     }
     
     Foreach ($Line in $ADResultArray) {
         If (!(($Line.EnableSkype -eq 'N') -or ($Line.EnableSkype -eq 'No'))) {
             Write-Output "Attempting to create Skype account for $($Line.DisplayName)"
             $SkypeResult = Enable-ContosoSkypeUser -UserPrincipalName $Line.UserPrincipalName
             If ($($SkypeResult.LineURI) -like "tel*") {
                 $ADPhoneNumber = $SkypeResult.LineURI.Split(':')[1]
                 $ADPhoneNumber = $ADPhoneNumber.Split(';')[0]
                 Set-Aduser -Identity $SkypeResult.Identity -OfficePhone $ADPhoneNumber
             }
             $SkypeResultArray = $SkypeResultArray + $SkypeResult
         }
     }
 
     Foreach ($Line in $ADResultArray) {
         If (!(($Line.EnableMailbox -eq 'N') -or ($Line.EnableMailbox -eq 'No'))) {
             Write-Output "Attempting to create mailbox for $($Line.DisplayName)"
             Enable-ContosoMailbox -UserPrincipalName $Line.UserPrincipalName
         }
     }
 
     Invoke-DirSync
 
     Foreach ($Line in $ADResultArray) {
        If (!(($Line.EnableMailbox -eq 'N') -or ($Line.EnableMailbox -eq 'No'))) {
             Write-Output "Attempting to provision O365 License for $($Line.DisplayName)"
             $Country = $Line.OfficeCode.SubString(0,2)            
             Enable-ContosoO365License -UserPrincipalName $Line.UserPrincipalName -Country $Country
         }
     }
     
     Write-Host "Pausing for 30 seconds to allow Exchange Online to provision mailboxes" -ForegroundColor Cyan
     Start-Sleep -Seconds 30
 
     Foreach ($Line in $SkypeResultArray) {
         If ($($Line.LineURI) -like "tel*") {
             Write-Output "Attempting to create UM Mailbox for $($Line.DisplayName)"
             $Upn = $Line.SipAddress.Split(':')[1]
             $Office = $Line.DialPlan.Split('.')[0]
             Enable-ContosoUMMailbox -UserPrincipalName $Upn -Office $Office
         }
     }
 
     Foreach ($Line in $ADResultArray) {
         $SIP = "sip:" + $($Line.UserPrincipalName)
         $SkypeResultLookup = $SkypeResultArray | Where-Object {$_.SipAddress -eq $SIP}
         If ($SkypeResultLookup.LineURI -like "tel*") {
             $PhoneNumber = $SkypeResultLookup.LineURI.Split(':')[1]
             $PhoneNumber = $PhoneNumber.Split(';')[0]
             $Line.SkypeNumber = $PhoneNumber
         }
     }
 
     Foreach ($Line in $ADResultArray) {
         If (($Line.SRNumber -match "^SR*") -and ($Line.SkypeNumber -match "\d{4,}")) {
             Update-SCSMStarterTicket -SRNumber $Line.SRNumber -UserPrincipalName $Line.UserPrincipalName -SAMAccountName $Line.SamAccountName -SkypeNumber $Line.SkypeNumber
         }
         ElseIf ($Line.SRNumber -match "^SR*") {
             Update-SCSMStarterTicket -SRNumber $Line.SRNumber -UserPrincipalName $Line.UserPrincipalName -SAMAccountName $Line.SamAccountName
         }
 
     }
 
     $defaultProperties = @('DisplayName','EmployeeID','SamAccountName','UserPrincipalName','SkypeNumber','Password')
     $defaultDisplayPropertySet = New-Object System.Management.Automation.PSPropertySet(�DefaultDisplayPropertySet�,[string[]]$defaultProperties)
     $PSStandardMembers = [System.Management.Automation.PSMemberInfo[]]@($defaultDisplayPropertySet)
     $ADResultArray | Add-Member MemberSet PSStandardMembers $PSStandardMembers
     
     $ExecutionStop = Get-Date
     $Timetaken = $ExecutionStop - $ExecutionStart
     $TimetakenInMin = "{0:N0}" -f ($Timetaken.TotalMinutes)
     Write-Output "Script completed in $TimetakenInMin minutes`n"
     Get-PSSession | Remove-PSSession -Confirm:$False
     return $ADResultArray
 }
 
 Function New-ContosoADUser {
  .SYNOPSIS 
     Creates a new AD account for a starter given the necessary parameters.
  .DESCRIPTION 
     This function creates a new AD user account for a starter based on the parameters defined below.  The function will automatically generate a
     unique SAM Account and UPN and will copy an existing template if available.  Of note only specific values for EmployeeType and Company are accepted.
  .NOTES 
     Author  : TheDudeAbides
     Requires: PowerShell Version 3.0, Active Directory PowerShell module
  .EXAMPLE
     .\New-ContosoAdUser -GivenName Robert -Surname Public -Initial Q -PreferredGivenName Bob -EmployeeID 123456 -EmployeeType F -Description "Job Site Manager" -BU 12345 -OfficeCode "US-DEN-1"
  .PARAMETER GivenName
     Required parameter for the first or given name of the user to be created
  .PARAMETER Surname
     Required parameter for the last name or names of the user to be created.  The function will automatically remove any spaces from the last name or apostrophes if found.
  .PARAMETER Initial
     Optional parameter for the middle initial of the user.  This should only be the first character of the middle name as applicable.
  .PARAMETER PreferredGivenName
     Optional parameter for the prefferred given name of the user if supplied.  This will be used in the Display name for the account.
  .PARAMETER EmployeeID
     Optional parameter for the employee ID of the user.  If not supplied, it will be filled in by FIM at a later time or automatically filled in if the user
     is a temp, contractor, or JV user.
  .PARAMETER EmployeeType
     Required parameter for the employee type, should be specified as either (F)ull-time, (T)emporary, (C)ontractor, or (JV).
  .PARAMETER Description
     Optional parameter for the description on the user.  This field can include details of the account, requestor, or Helpdesk ticket to help document the user
     to be created.
  .PARAMETER BU
     Optional parameter for the business unit of the account to be created.
  .PARAMETER OfficeCode
     Optional parameter for the Office code of the account to be created.
  .PARAMETER Company
     Required parameter for the company name of the account to be created.  Accepted values are Contoso, TailSpinToys, JV and the company name is used to generate
     the correct UPN and email address.
 #>
 [cmdletbinding()]
     Param(
         [Parameter(Position=0,Mandatory=$True,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
         [string]$GivenName="",
         [Parameter(Position=1,Mandatory=$True,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
         [string]$Surname="",
         [Parameter(Position=2,Mandatory=$False,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
         [string]$Initial="",
         [Parameter(Position=3,Mandatory=$False,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
         [string]$PreferredGivenName="",
         [Parameter(Position=4,Mandatory=$False,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
         [string]$EmployeeID="",
         [Parameter(Position=5,Mandatory=$True,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)][ValidateSet('F','T','C','JV','-')]
         [string]$EmployeeType="",
         [Parameter(Position=6,Mandatory=$False,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
         [string]$Description="",
         [Parameter(Position=7,Mandatory=$False,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)][ValidateRange(00000,99999)]
         [int]$BU="",
         [Parameter(Position=8,Mandatory=$False,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
         [string]$OfficeCode="",
         [Parameter(Position=9,Mandatory=$False,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)][ValidateSet('Contoso','TailSpinToys','JV')]
         [string]$Company="Contoso"
     )
 
     $StarterObject = New-Object -TypeName psobject -Property @{
         EmployeeID = "$EmployeeID"
         OfficeCode = "$OfficeCode"
         BU = $BU
         Surname = "$Surname"
         PreferredGivenName = "$PreferredGivenName"
         GivenName = "$GivenName"
         Initial = "$Initial"
         Company = "$Company"
         EmployeeType = "$EmployeeType"
         Description = "$Description"
     }
     Try {
         $UserResult = $StarterObject | Confirm-ValidUser
         $UserResult.EnableMailbox = $null
         $UserResult.EnableSkype = $null
         $UserResult.SRNumber = $null
     }
     Catch {
         $ErrorMessage = $_.Exception.Message
         $FailedItem = $_.Exception.ItemName
         exit
     }
 
     $CleanedSurname = $UserResult.Surname.replace(" ","")
     $CleanedSurname = $UserResult.Surname.replace("'","")
     If ($line.PreferredGivenName -match "\w+") {
             $FullName = $UserResult.PreferredGivenName + " " + $UserResult.Surname
     }
     Else {
             $FullName = $UserResult.GivenName + " " + $UserResult.Surname
     }
     $Country = $UserResult.OfficeCode.Substring(0,2)
     $Region = $RegionMappings.$Country
     
     Try {
         $PotentialSAMAccountName = Get-UniqueSAMAccountName -GivenName $UserResult.GivenName -Surname $CleanedSurname -Initial $Initial
     } 
     Catch {
         throw 'Could not generate a unique SAMAccountName'
     }
     
     Try {
         $PotentialUPN = Get-UniqueUPN -GivenName $UserResult.GivenName -Surname $CleanedSurname -Initial $UserResult.Initial -Company $UserResult.Company
     } 
     Catch {
         throw 'Could not generate a unique UserPrincipalName'
     }
 
     $Password = Get-RandomPassword
     $SecurePW = ConvertTo-SecureString $Password -AsPlainText -Force
     $Searchbase = "OU=Users & Groups,dc=Contoso,dc=com"
     $NewADUserParams = @{
         Name=$FullName
         SamAccountName=$PotentialSAMAccountName
         UserPrincipalName=$PotentialUPN
         GivenName=$UserResult.GivenName
         Surname=$UserResult.Surname
         Instance=$null
         EmployeeID=$UserResult.EmployeeID
         Company=$UserResult.Company
         Department=$UserResult.BU
         Description=$UserResult.Description
         DisplayName=$FullName
         Initials=$UserResult.Initial
         Office=$UserResult.OfficeCode
         AccountPassword=$SecurePW
         Enabled=$false
         ChangePasswordAtLogon=$True
         Path=$Null
         EmailAddress=$Null
         OtherAttributes = @{EmployeeType = ($UserResult.EmployeeType)};
     }
 
     If ($UserResult.EmployeeType -match "^(F|T)$") {
         $OfficeCodeSplit = $OfficeCode.Split("-")
         $OfficeCodeSearchterms = (($OfficeCode),($OfficeCode.Replace("-","")),($OfficeCodeSplit[0]+"-"+$OfficeCodeSplit[1]+$OfficeCodeSplit[2]),($OfficeCodeSplit[0]+$OfficeCodeSplit[1]+"-"+$OfficeCodeSplit[2]))
         $AllActiveDirectoryTemplates = Get-ADUser -Filter {Name -like "*Template*"}
         Foreach ($term in $OfficeCodeSearchterms) {
             If ($AllActiveDirectoryTemplates -match $term) {
                 $Instance = $AllActiveDirectoryTemplates -match $term
                 $DN = $Instance.DistinguishedName.Split(',')
                 $NewADUserParams.Path = ($DN[1..($DN.Length-1)]) -join ','
                 $NewADUserParams.Instance = $($Instance)
             }
         }
         If ($NewADUserParams.Path -eq $null) {
             $NewADUserParams.Path = (Get-ADOrganizationalUnit -SearchBase $Searchbase -SearchScope OneLevel -Filter * -Properties Description | Where-Object {$_.DistinguishedName -like "*$Region*"}).DistinguishedName
         }
     }
     Elseif ($EmployeeType -eq "C") {
         $NewADUserParams.Path = (Get-ADOrganizationalUnit -SearchBase $Searchbase -Filter {(Description -eq "Default Consultants")} -Properties Description | Where-Object {$_.DistinguishedName -like "*$Region*"}).DistinguishedName | Out-String
     }
     Elseif ($EmployeeType -eq "JV") {
         $NewADUserParams.Path = "OU=_External,OU=Microsoft Exchange Contacts,OU=Domain,OU=Users & Groups,DC=Contoso,DC=com"
     }
 
     If ($NewADUserParams.Instance -eq $null) {
         $NewADUserParams.Remove('Instance')
     }
     
     Try {
         $NewAccount = New-ADUser @NewADUserParams -PassThru -ErrorAction Stop
 
         $GLOBAL:AllUPNs = $GLOBAL:AllUPNs + $PotentialUPN
         $GLOBAL:AllSAMAccountNames = $GLOBAL:AllSAMAccountNames + $PotentialSAMAccountName
     }
     Catch {
         $ErrorMessage = $_.Exception.Message
         $FailedItem = $_.Exception.ItemName
         throw "Account creation failed, error message is $ErrorMessage"
     }
 
     $UserMembership = (Get-ADUser $NewAccount -Properties memberof).memberof
     $UserMembership | Remove-ADGroupMember -Members $NewAccount.DistinguishedName -Confirm:$False
 
     If ($Region -eq 'AMER') {
         Add-ADGroupMember -Identity "AMERICAS-Users" -Members $NewAccount
         Add-ADGroupMember -Identity "Contoso-Users" -Members $NewAccount
         $OfficeGroup = "AM-Office-" + $UserResult.OfficeCode
         Add-ADGroupMember -Identity $OfficeGroup -Members $NewAccount -ErrorAction SilentlyContinue
     } 
     Elseif ($Region -eq 'EMEA') {
         If ($EmpType -eq "P") {
             Add-ADGroupMember -Identity "EMEAI-CITRIX" -Members $NewAccount
             
             Add-ADGroupMember -Identity "EMEAI-SSL-General" -Members $NewAccount
             $OfficeGroup = "EMEAI-Office-" + $UserResult.OfficeCode
             Add-ADGroupMember -Identity $OfficeGroup -Members $NewAccount -ErrorAction SilentlyContinue
         }
         Elseif ($EmpType -eq "C") {
             Add-ADGroupMember -Identity "EMEAI-Office-External" -Members $NewAccount
         }
         Add-ADGroupMember -Identity "Contoso-Users" -Members $NewAccount
         Add-ADGroupMember -Identity "EMEAI Users" -Members $NewAccount
     }
     Elseif ($Region -eq 'ASIA') {
         Add-ADGroupMember -Identity "ASIA Users" -Members $NewAccount
         Add-ADGroupMember -Identity "Contoso-Users" -Members $NewAccount
         If (($Country -eq 'CN') -or ($Country -eq 'TW')) {
             $OfficeGroup = $UserResult.OfficeCode.Replace('-','') + "Users"
             Add-ADGroupMember -Identity $OfficeGroup -Members $NewAccount -ErrorAction SilentlyContinue
         }
         Else {
             $ShortOfficeCode = $OfficeCode.Replace("-","")
             $OfficeGroup = "ASIA-Office-" + $ShortOfficeCode
             Add-ADGroupMember -Identity $OfficeGroup -Members $NewAccount -ErrorAction SilentlyContinue
         }
     }
 
     $FinishedAccount = New-Object -TypeName psobject -Property @{
         Name=$FullName
         SamAccountName=$PotentialSAMAccountName
         UserPrincipalName=$PotentialUPN
         GivenName=$UserResult.GivenName
         Surname=$UserResult.Surname
         TemplateUsed=$Instance.Name
         EmployeeID=$UserResult.EmployeeID
         Company=$UserResult.Com
`

