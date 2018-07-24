---
Author: jon webster
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 830
Published Date: 
Archived Date: 2009-02-04t17
---

# enable-mailboxjunkemail. - enable-mailboxjunkemail

## Description

enable server side junk e-mail filtering for exchange 2007 mailboxes

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 
 param(
 	$Identity,
 	[string]$CasURL,
 	[string]$User,
 	[string]$Password,
 	$DomainController,
 	[switch]$help
 )
 	
 BEGIN
 {
 	Function Usage
 	{
 		Write-Host @'
 Enable Server Side Junk E-mail filtering through Outlook Web Access
 `n
 Usage:
 	Enable-MailboxJunkEmail -Identity <string> -CasURL <string> -User <string> -Password <string> -DomainController <string>
 `n
 Parameters:
 	Identity: The Identity of mailbox who's server side Junk E-mail you want to enable.
 	CasURL: The (OWA) Client Access Server Address of the exchange server you want to use.
 	User: The fullaccess user's 'username' connecting to the mailbox that you want to change.
 	Password: The fullaccess user's 'password' connecting to the mailbox that you want to change.
 	DomainController: The fully qualified domain name (FQDN) of the domain controller that you want to use.
 `n
 Example:
 	Enable-MailboxJunkEmail -Identity "john.doe@consoto.com" -CasURL "mail.consoto.com" -User "CONSOTO\Administrator" -Password "AdminPassword!"
 '@
 	}
 	
 	if($help -or $args -contains "-?")
 	{
 		Usage
 		return
 	}
 	
 	Function ValidateParams
 	{
 		$ErrorMessage = ""
 		
 		if(!$CasURL) {
 			$ErrorMessage += "Missing parameter: The -CasURL parameter is required. Please pass a valid Client Access Server Url`n"
 		}
 		
 		if(!$User) {
 			$ErrorMessage += "Missing parameter: The -User parameter is required. Please pass a valid Username for OWA mailbox authentication`n"
 		}
 		
 		if(!$Password) {
 			$ErrorMessage += "Missing parameter: The -Password parameter is required. Please pass a valid password for OWA mailbox authentication`n"
 		}
 		
 		if($ErrorMessage)
 		{
 			throw $ErrorMessage
 			break
 		}
 	}
 	
 	Function ValidatePipeline
 	{
 		if($_)
 		{
 			$ErrorMessage = ""
 			if(!$_.Identity)
 			{
 				$ErrorMessage += "Missing Pipeline property: The Identity property is required."
 			} else {
 				Set-Variable -Name Identity -Scope 1 -Value $_.Identity
 			}
 			
 			if($ErrorMessage)
 			{
 				throw $ErrorMessage
 				break
 			}
 		}
 	}
 
 	Function UpdateJunk
 	{
 		param ([string]$mbMailbox)
 	
 		$xmlstr = "<params><fEnbl>1</fEnbl></params>"
 		$req.Open("POST", "https://" + $CasURL + "/owa/" + $mbMailbox + "/ev.owa?oeh=1&ns=JunkEmail&ev=Enable", $False)
 		$req.setRequestHeader("Content-Type", "text/xml; charset=""UTF-8""")
 		$req.setRequestHeader("Content-Length", $xmlstr.Length)
 		$req.Send($xmlstr)
 
 		Write-Debug $req.status
 		Write-Debug $req.GetAllResponseHeaders()
 
 		if($req.status -ne 200)
 		{
 			Write-Error $req.responsetext
 			return
 		}
 	
 		if($req.responsetext -match "name=lngFrm")
 		{
 			Write-Host "Mailbox has not been logged onto before via OWA"
 	
 			$pattern = "<option selected value=""(.*?)"">"
 			$matches = [regex]::Matches($req.responsetext,$pattern)
 			if($matches.count -eq 2)
 			{
 				$lcidarry = $matches[0].Groups[1].Value
 				Write-Debug $lcidarry
 				$tzidarry = $matches[1].Groups[1].Value
 				Write-Debug $tzidarry
 				$pstring = "lcid=" + $lcidarry + "&tzid=" + $tzidarry
 				$req.Open("POST", "https://" + $CasURL + "/owa/" + $mbMailbox + "/lang.owa", $False)
 				$req.setRequestHeader("Content-Type", "application/x-www-form-urlencoded")
 				$req.setRequestHeader("Content-Length", $pstring.Length)
 				$req.send($pstring)
 				if($req.responsetext -match "errMsg")
 				{
 					Write-Error "Authentication Error"
 				} else {
 					Write-Debug $req.status
 					if($req.status -eq 200 -and $req.responsetext -notmatch "errMsg")
 					{ 
 						Write-Host "Trying to update the Junk E-mail setting again."
 						UpdateJunk($mbMailbox)
 						Write-Host "Removing OWA Language and Timezone settings..."
 
 						&{
 							$warningPreference = "Stop"
 							$script:count = 0
 							$loop = $true
 							while($loop)
 							{
 								$loop = $false
 								Set-Mailbox $mbMailbox -Languages $null -DomainController $DomainController
 								trap {
 									
 									if($script:count -lt 5)
 									{
 										Write-Debug "Unable to Reset Languages trying again in 5 seconds."
 										Sleep 5
 										Set-Variable -Name loop -Scope 1 -Value $true
 										$script:count++
 									} else { Write-Debug "Failed." }
 									continue
 								}
 							}
 							$warningPreference = "Continue"
 						}
 					} else {
 						Write-Warning "Failed to set Default OWA settings"
 					}
 				}
 			} else {
 				Write-Warning "Script failed to retrieve default values"
 			}
 		Write-Host "Junk E-Mail setting Changed Successfully"
 		}
 	}
 	ValidateParams
 }
 PROCESS
 {
 	ValidatePipeline
 
 	$mbx = Get-Mailbox -Identity $Identity -DomainController $DomainController -ErrorAction SilentlyContinue
 	if(!$mbx) {throw "Invalid Mailbox specified: $Identity"}
 	
 	$szXml = "destination=https://" + $CasURL + "/owa/&flags=0&username=" + $User
 	$szXml = $szXml + "&password=" + $Password + "&SubmitCreds=Log On&forcedownlevel=0&trusted=0"
 	
 	$req = New-Object -COMObject "MSXML2.ServerXMLHTTP.6.0"
 	$req.Open("POST", "https://" + $CasURL + "/owa/auth/owaauth.dll", $False)
 	$req.SetOption(2, 13056)
 	$req.Send($szXml)
 	
 	Write-Debug $req.GetAllResponseHeaders()
 	if($req.responsetext -match "wrng")
 	{
 		Write-Error "The user name or password that you entered is not valid. Try entering it again."
 		return;
 	}
 
 	UpdateJunk($mbx.PrimarySMTPAddress)	
 }
`

