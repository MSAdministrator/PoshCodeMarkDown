---
Author: will steele
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6650
Published Date: 2017-12-13t15
Archived Date: 2017-03-03t11
---

# read gmail pop - 

## Description

this script is a proof of concept. further work needs to be done.  it requires the user to enter a valid username and password for a gmail.com account.  it then attempts to form an ssl connection with the server, and, retrieve the first email. unfortunately it returns random results.  perhaps someone can improve upon it with some more sockets knowledge than i have.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 	.AUTHOR
 		Will Steele (wlsteele@gmail.com)
 
 	.DEPENDENCIES
 		Powershell v2 (for Out-Gridview cmdlet)
 
 	.DESCRIPTION
 		This script is a proof of concept. Further work needs to be done.  It 
 		requires the user to enter a valid username and password for a gmail.com 
 		account.  It then attempts to form an SSL connection with the server, and, 
 		retrieve the first email. Unfortunately it returns random results.  Perhaps 
 		someone can improve upon it.
 	
 	.EXAMPLE
 		Get-Gmail -username 'you@gmail.com' -password '\/0u|>p@55w0rd'
 	
 	.EXTERNALHELP
 		None.
 		
 	.FORWARDHELPTARGETNAME
 		None.
 		
 	.INPUTS
 		System.Object
 		
 	.LINK
 		http://learningpcs.blogspot.com/2012/01/powershell-v2-read-gmail-more-proof-of.html.
 		
 	.NAME
 		Get-Gmail.ps1
 		
 	.NOTES
 		The script is further explored to find 'From' addressses in the link above.
 		I have piped the $str variable to Out-Gridview. This can be changed to something
 		more suitable for real processing.
 		
 	.OUTPUTS
 		System.String
 		
 	.PARAMETER username
 		A required parameter for a valid gmail email user. Use the whole string.
 		
 	.PARAMETER password
 		A required parameter for the account password.
 	
 	.SYNOPSIS
 		Read .
 #>
 
 [CmdletBinding()]
 param(
 	[Parameter(
 		Mandatory = $true,
 		Position = 0,
 		ValueFromPipeline = $true
 	)]
 	[ValidateNotNullOrEmpty()]
 	[String]
 	$username,
 	
 	[Parameter(
 		Mandatory = $true,
 		Position = 1,
 		ValueFromPipeline = $true
 	)]
 	[ValidateNotNullOrEmpty()]
 	[String]
 	$password
 )
 
 Clear-Host 
 
 try {
 	Write-Output "Creating new TcpClient."
 	$tcpClient = New-Object -TypeName System.Net.Sockets.TcpClient
 	
 	$tcpClient.Connect("pop.gmail.com", 995)
 	
 	if($tcpClient.Connected) {
 		Write-Output "You are connected to the host. Attempting to get SSL stream."
 		
 		Write-Output "Getting SSL stream."
 		[System.Net.Security.SslStream] $sslStream = $tcpClient.GetStream()
 		
 		Write-Output "Authenticating as client."		
 		$sslStream.AuthenticateAsClient("pop.gmail.com");
 		
 		if($sslStream.IsAuthenticated) {
 			Write-Output "You have authenticated. Attempting to login."
 			[System.IO.StreamWriter] $sw = $sslstream
 			
 			[System.IO.StreamReader] $reader = $sslstream
 			
 			$sw.WriteLine("USER $username")
 			
 			$sw.Flush()
 			
 			$sw.WriteLine("PASS $password");
 			$sw.Flush()
 			
 			$sw.WriteLine("RETR 1")
 			$sw.Flush()
 			
 			$sw.WriteLine("Quit ");
 			$sw.Flush();
 			
 			[String] $str = [String]::Empty
 			[String] $strTemp = [String]::Empty
 
 			while (($strTemp = $reader.ReadLine()) -ne $null) {
 				if($strTemp -eq '.') {
 					break;
 				}
 
 				if ($strTemp.IndexOf('-ERR') -ne -1) {
 					break;
 				}
 					
 				$str += $strTemp;
 			}
 			
 			Write-Output "`nOutput email"
 			$str | Out-GridView
 		} else { 
 			Write-Error "You were not authenticated. Quitting."
 		}
 		
 	} else {
 		Write-Error "You are not connected to the host. Quitting"
 	}
 }
 
 catch {
 	$_
 }
 
 finally {
 	Write-Output "Script complete."
 }
`

