---
Author: rich kusak
Publisher: 
Copyright: 
Email: 
Version: 2.0.0.0
Encoding: ascii
License: cc0
PoshCode ID: 2947
Published Date: 2011-09-07t14
Archived Date: 2011-11-05t17
---

# resetoutlooklivepassword - 

## Description

the reset-outlooklivepassword function resets an outlook live (live@edu) acccount password.

## Comments



## Usage



## TODO



## function

`reset-outlooklivepassword`

## Code

`#
 #
 function Reset-OutlookLivePassword {
 <#
 	.SYNOPSIS
 		Resets an Outlook Live account password.
 
 	.DESCRIPTION
 		The Reset-OutlookLivePassword function resets an Outlook Live (Live@edu) acccount password.
 		A remote session is opened to the Outlook Live service. Connecting to the remote service requires administrative credentials to
 		Outlook Live. The function optionally stores the credential for subsequent use with the SaveCredential parameter.
 		The stored credential is protected using the Windows Data Protection API (DPAPI).
 
 	.PARAMETER Identity
 		Specifies the identity of the Outlook Live user account. Acceptable values include:
 		
 		* User principal name (UPN)
 
 	.PARAMETER Password
 		Specifies a new password for the account.
 		A plain text string may be supplied or by not specifing the parameter a prompt for secure entry will appear.
 
 	.PARAMETER ResetPasswordOnNextLogon
 		Specifies that the user will be forced to reset their password at next logon.
 	
 	.PARAMETER Credential
 		Specifies a credential for connecting to the Outlook Live remote session.
 
 	.PARAMETER SaveCredential
 		Specifies that the credential provided to connect to the Outlook Live remote session will be saved for subsequent use.
 		The Windows Data Protection API (DPAPI) is used to encrypt the password string representation.
 
 	.EXAMPLE
 		Reset-OutlookLivePassword unique.name@mydomain.edu
 		Resets the Outlook Live password on the account unique.name@mydomain.edu to the password specified in a secure entry prompt.
 
 	.EXAMPLE
 		Reset-OutlookLivePassword unique.name@mydomain.edu -Password HelloDave
 		Resets the Outlook Live password on the account unique.name@mydomain.edu to the password HelloDave.
 
 	.EXAMPLE
 		Reset-OutlookLivePassword unique.name@mydomain.edu -Password HelloDave -SaveCredential
 		Resets the Outlook Live password on the account unique.name@mydomain.edu to the password HelloDave and saves the remote session credential.
 
 	.INPUTS
 		System.String, System.Security.SecureString
 
 	.OUTPUTS
 		None
 
 	.NOTES
 		Name: Reset-OutlookLivePassword
 		Author: Rich Kusak
 		Created: 2010-07-30
 		LastEdit: 2011-08-28 16:27
 		Version: 2.0.0.0
 
 	.LINK
 		http://outlookliveanswers.com/forums/p/10/15.aspx
 
 #>
 
 	[CmdletBinding(DefaultParameterSetName='Default', SupportsShouldProcess=$true)]
 	param (
 		[Parameter(Position=0, Mandatory=$true, ValueFromPipeline=$true)]
 		[string]$Identity,
 		
 		[Parameter(ValueFromPipelineByPropertyName=$true)]
 		[ValidateScript({
 			if ($_ -is [System.String] -or $_ -is [System.Security.SecureString]) {$true} else {
 				throw "The argument '$_' is not of type System.String or System.Security.SecureString."
 			}
 		})]
 		$Password = (Read-Host 'Enter New Password' -AsSecureString),
 		
 		[Parameter(ValueFromPipelineByPropertyName=$true)]
 		[switch]$ResetPasswordOnNextLogon,
 		
 		[Parameter(ParameterSetName='Default', ValueFromPipelineByPropertyName=$true)]
 		[Parameter(ParameterSetName='Save', Mandatory=$true, ValueFromPipelineByPropertyName=$true)]
 		[System.Management.Automation.PSCredential]$Credential,
 		
 		[Parameter(ParameterSetName='Save', ValueFromPipelineByPropertyName=$true)]
 		[switch]$SaveCredential
 	)
 	
 	begin {
 		
 		function Save-Credential {
 			param ($Path, $Credential)
 			
 			if (Test-Path $Path) {
 				$attribs = [IO.FileAttributes]('Normal')
 				Set-ItemProperty $Path -Name Attributes -Value $attribs -Force
 			}
 			
 			New-Object PSObject -Property @{
 				'UserName' = $Credential.UserName
 				'Password' = $Credential.Password | ConvertFrom-SecureString
 			} | Export-Csv $Path -NoTypeInformation -Force
 
 			$attribs = [IO.FileAttributes]('Hidden', 'ReadOnly')
 			Set-ItemProperty $Path -Name Attributes -Value $attribs -Force
 		
 		$psFolder = Split-Path $PROFILE
 		$credFile = Join-Path $psFolder -ChildPath 'b94437da-2522-4bd4-903f-4b3f8ec7132a.csv'
 		
 		if (Test-Path $credFile) {
 			if ($Credential -and $SaveCredential) {
 				Write-Warning 'A saved credential already exists.'
 
 				do {
 					$prompt = Read-Host 'Do you want to overwrite the saved credential? (Y)es or (N)o.'
 				} until ('^Yes$|^No$' -match $prompt)
 			}
 
 			switch ($prompt) {
 				{$_ -match 'Y'} {
 					Write-Debug "Performing operation 'Save-Credential' on target '$credFile'."
 					Save-Credential -Path $credFile -Credential $Credential
 					$importedCredFile = Import-Csv $credFile
 					$liveUserName = $importedCredFile.UserName
 					$livePassword = $importedCredFile.Password | ConvertTo-SecureString
 					break
 				}
 
 				{$_ -match 'N'} {
 					$liveUserName = $Credential.UserName
 					$livePassword = $Credential.Password
 					break
 				}
 
 				default {
 					$importedCredFile = Import-Csv $credFile
 					$liveUserName = $importedCredFile.UserName
 					$livePassword = $importedCredFile.Password | ConvertTo-SecureString
 				}
 			
 			$liveCred = New-Object System.Management.Automation.PSCredential($liveUserName, $livePassword)
 
 		} else {
 			if (-not $Credential) {
 				try {
 					$Credential = Get-Credential -ErrorAction Stop
 				} catch {
 					throw $_
 				}
 			}
 			
 			$liveCred = New-Object System.Management.Automation.PSCredential($Credential.UserName, $Credential.Password)
 
 			if ($SaveCredential) {
 				if (-not (Test-Path $psFolder)) {
 					New-Item $psFolder -ItemType Directory -Force | Out-Null
 				}
 
 				Write-Debug "Performing operation 'Save-Credential' on target '$credFile'."
 				Save-Credential -Path $credFile -Credential $Credential
 			}
 		}
 		
 		$connectionUri = 'https://ps.outlook.com/powershell/'
 		$newPSSessionParameters = @{
 			'ConfigurationName' = 'Microsoft.Exchange'
 			'ConnectionUri' = $connectionUri
 			'Credential' = $liveCred
 			'Authentication' = 'Basic'
 			'AllowRedirection' = $true
 		}
 		
 		try {
 			Write-Debug "Performing operation 'New-PSSession' on target '$connectionUri'."
 			$session = New-PSSession @newPSSessionParameters -ErrorAction Stop
 			Write-Verbose "Successfully opened new remote session to '$connectionUri'."
 			if ($PSBoundParameters['Verbose']) {
 				$session
 				Write-Host
 			}
 		} catch {
 			throw $_
 		}
 		
 	
 	process {
 		
 		if ($Password -is [System.String]) {
 			Write-Debug "Performing operation 'ConvertTo-SecureString' on target 'Password'."
 			$Password = $Password | ConvertTo-SecureString -AsPlainText -Force
 		}
 		
 		if ($PSBoundParameters['WarningAction']) {
 			$WarningAction = $PSBoundParameters['WarningAction']
 		} else {
 			$WarningAction = $WarningPreference
 		}
 			
 		if ($PSBoundParameters['WhatIf']) {
 			$WhatIf = $PSBoundParameters['WhatIf']
 		} else {
 			$WhatIf = $WhatIfPreference
 		}
 		
 		$setMailboxParameters = @{
 			'Identity' = $Identity
 			'Password' = $Password
 			'ResetPasswordOnNextLogon' = $ResetPasswordOnNextLogon
 			'WarningAction' = $WarningAction
 			'WhatIf' = $WhatIf
 		}
 
 		try {
 			Write-Debug "Performing operation 'Invoke-Command' on target '$($session.ComputerName)'."
 			Invoke-Command -Session $session -ArgumentList $setMailboxParameters -ErrorAction Stop -ScriptBlock {
 				param ($setMailboxParameters)
 				Set-Mailbox @setMailboxParameters
 			}
 
 		} catch {
 			return Write-Error $_
 		}
 	
 	end {
 		
 		Write-Debug "Performing operation 'Remove-PSSession' on target '$($session.ComputerName)'."
 		Remove-PSSession $session -ErrorAction Stop -WhatIf:$false
 		Write-Verbose "Successfully removed remote session to '$connectionUri'."
`

