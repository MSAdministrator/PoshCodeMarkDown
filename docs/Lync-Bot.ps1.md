---
Author: michael diarmid
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5222
Published Date: 2017-06-05t17
Archived Date: 2017-05-16t03
---

# lync bot - 

## Description

a simple lync bot in powershell that will respond to set commands etc, was bored today.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 	Powershell Lync Bot
 	This will respond to the commands below in the switch statement.
 
 	Just a simple script to see how it could be done, took a lot of digging
 	as it seems no-one has done this before?
 
 	***Powershell window needs to remain open for events to fire***
 
 	Author: Michael Diarmid
 	Email: michael.diarmid@atos.net
 
 #>
 
 Get-EventSubscriber | Unregister-Event
 
 $ModelPath = "C:\Program Files (x86)\Microsoft Lync\SDK\Assemblies\Desktop\Microsoft.Lync.Model.DLL"
 
 If (Test-Path $ModelPath)
 {
 	Import-Module $ModelPath
 }
 Else
 {
 	Write-Host 'Please Import the "Microsoft.Lync.Model.DLL" Before using Command'
 	break
 }
 
 $client = [Microsoft.Lync.Model.LyncClient]::GetClient()
 
 $global:action = {
 	$Conversation = $Event.Sender.Conversation
 	
 	$msg = New-Object "System.Collections.Generic.Dictionary[Microsoft.Lync.Model.Conversation.InstantMessageContentType,String]"
 	
 	$Modality = $Conversation.Modalities[1]
 	
 	[string]$msgStr = $Event.SourceArgs.Text
 	$msgStr = $msgStr.ToString().ToLower().Trim()
 	
 	switch ($msgStr)
 	{
 		"hey" {
 			$msg.Add(1, '<b>hello</b>')
 			
 		}
 		"hi" {
 			$msg.Add(1, '<i>Hello</i>')
 			
 		}
 		"hello" {
 			$msg.Add(1, '<u>hey</u>')
 			
 		}
 		"moo" {
 			$msg.Add(0, 'Are you a cow?')
 			
 		}
 		"help" {
 			$msg.Add(0, 'Mr bot at your service!')
 			$msg.Add(1, '<b>Available commands:</b>')
 			$msg.Add(1, 'hi, hey, hello')
 			$msg.Add(1, 'moo')
 			$msg.Add(1, 'time')						
 		}
 		"time" {
 			$now = Get-Date
 			$msg.Add(0, '' + $now)
 		}
 		default
 		{
 				$sendMe = 0
 		}
 	}
 	
 	if ($sendMe -eq 1)
 	{		
 		$null = $Modality.BeginSendMessage($msg, $null, $msg)
 	}
 }
 
 foreach ($con in $client.ConversationManager.Conversations)
 {
 	$moo = $con.Participants | Where { !$_.IsSelf }
 
 	foreach ($mo in $moo)
 	{
 		$mo.Contact.uri
 		if (!(Get-EventSubscriber $mo.Contact.uri))
 		{
 			Register-ObjectEvent -InputObject $mo.Modalities[1] -EventName "InstantMessageReceived" -SourceIdentifier $mo.Contact.uri -action $global:action
 		}
 	}
 }
 
 $conversationMgr = $client.ConversationManager
 Register-ObjectEvent -InputObject $conversationMgr `
 					 -EventName "ConversationAdded" `
 					 -SourceIdentifier "NewIncomingConversation" `
 					 -action {
 	$client = [Microsoft.Lync.Model.LyncClient]::GetClient()
 	foreach ($con in $client.ConversationManager.Conversations)
 	{
 		$moo = $con.Participants | Where { !$_.IsSelf }
 		foreach ($mo in $moo)
 		{
 			#$mo.Contact.uri
 			if (!(Get-EventSubscriber $mo.Contact.uri))
 			{
 				Register-ObjectEvent -InputObject $mo.Modalities[1] -EventName "InstantMessageReceived" -SourceIdentifier $mo.Contact.uri -action $global:action
 			}
 		}
 	}
 	
 }
 
 
 
 #$con.BeginSetProperty([Microsoft.Lync.Model.Conversation.ConversationProperty]::Subject, "A title woo", $null, $null)
`

