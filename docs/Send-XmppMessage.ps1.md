---
Author: halr9000
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 103
Published Date: 2008-01-07t13
Archived Date: 2017-02-21t07
---

# send-xmppmessage - 

## Description

sends an xmpp (jabber) instant message.  these parameters are mandatory

## Comments



## Usage



## TODO



## function

`send-xmppmessage`

## Code

`#
 #
 function Send-XmppMessage {
 	param (
 		$From = $( Throw "You must specify a Jabber ID for the sender." ),
 		$To = $( Throw "You must specify a Jabber ID for the recipient." ),
 		$Body = $( Throw "You must specify a body for the message." )
 	)
 	
 	function Read-HostMasked( [string]$prompt="Password" ) {
 		$password = Read-Host -AsSecureString $prompt; 
 		$BSTR = [System.Runtime.InteropServices.marshal]::SecureStringToBSTR($password);
 		$password = [System.Runtime.InteropServices.marshal]::PtrToStringAuto($BSTR);
 		[System.Runtime.InteropServices.Marshal]::ZeroFreeBSTR($BSTR);
 		return $password;
 	}
 	$assemblyPath = $(resolve-path $profiledir\Assemblies\agsXMPP.dll)
 	[void][reflection.assembly]::LoadFrom( $assemblyPath )
 	$jidSender 		= New-Object agsxmpp.jid( $From )
 	$jidReceiver 	= New-Object agsxmpp.jid ( $To )
 	$xmppClient 	= New-Object agsxmpp.XmppClientConnection( $jidSender.Server )
 	$Message 		= New-Object agsXMPP.protocol.client.Message( $jidReceiver, $Body )
 	
 	#$xmppClient.UseSSL 					= $FALSE
 	#$xmppClient.UseStartTLS 				= $FALSE
 	
 	$xmppClient.AutoAgents 					= $FALSE
 	$xmppClient.AutoRoster 					= $FALSE
 	
 	$xmppClient.AutoResolveConnectServer 	= $TRUE
 	if ( !$password ) { $password = Read-HostMasked }
 	
 	$xmppClient.Open( $jidSender.User, $Password )
 		while ( !$xmppClient.Authenticated ) {
 			Write-Verbose $xmppClient.XmppConnectionState
 			Start-Sleep 1
 		}
 	#$xmppClient.SendMyPresence()
 	$xmppClient.Send( $Message )
 	Start-Sleep 1
 	$xmppClient.Close()
 }
`

