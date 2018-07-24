---
Author: joerg hochwald
Publisher: 
Copyright: 
Email: 
Version: 6.1
Encoding: ascii
License: cc0
PoshCode ID: 6432
Published Date: 2016-07-01t04
Archived Date: 2016-07-04t03
---

# send-hipchat - 

## Description

send a notification message to a hipchat room via a restful call to the hipchat api v2 atlassian requires a separate token for each room within hipchat!

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 
 
 <#
 		#################################################
 		#################################################
 
 		Support: https://github.com/jhochwald/NETX/issues
 #>
 
 
 
 <#
 		Copyright (c) 2012-2016, NET-Experts <http:/www.net-experts.net>.
 		All rights reserved.
 
 		Redistribution and use in source and binary forms, with or without
 		modification, are permitted provided that the following conditions are met:
 
 		1. Redistributions of source code must retain the above copyright notice,
 		this list of conditions and the following disclaimer.
 
 		2. Redistributions in binary form must reproduce the above copyright notice,
 		this list of conditions and the following disclaimer in the documentation
 		and/or other materials provided with the distribution.
 
 		3. Neither the name of the copyright holder nor the names of its
 		contributors may be used to endorse or promote products derived from
 		this software without specific prior written permission.
 
 		THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 		AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 		IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 		ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
 		LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 		CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 		SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 		INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
 		CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 		ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF
 		THE POSSIBILITY OF SUCH DAMAGE.
 
 		By using the Software, you agree to the License, Terms and Conditions above!
 #>
 
 
 function global:Send-HipChat {
 	<#
 			.SYNOPSIS
 			Send a notification message to a HipChat room.
 
 			.DESCRIPTION
 			Send a notification message to a HipChat room via a RESTful Call to
 			the HipChat API V2 Atlassian requires a separate token for each room
 			within HipChat!
 
 			So please note, that the Room and the Token parameter must match.
 
 			.PARAMETER Token
 			HipChat Auth Token
 
 			.PARAMETER Room
 			HipChat Room Name that get the notification.
 			The Token has to fit to the Room!
 
 			.PARAMETER notify
 			Whether this message should trigger a user notification
 			(change the tab color, play a sound, notify mobile phones, etc).
 			Each recipient's notification preferences are taken into account.
 
 			.PARAMETER color
 			Background color for message.
 
 			Valid is
 			- yellow
 			- green
 			- red
 			- purple
 			- gray
 			-random
 
 			.PARAMETER Message
 			The message body itself. Please see the HipChat API V2 documentation
 
 			.PARAMETER Format
 			Determines how the message is treated by our server and rendered
 			inside HipChat applications
 
 			.EXAMPLE
 			PS C:\> Send-HipChat -Message "This is just a BuildServer Test" -color "gray" -Room "Testing" -notify $true
 
 			Description
 			-----------
 			Sent a HipChat Room notification "This is just a BuildServer Test" to
 			the Room "Testing".
 			It uses the Color "gray", and it sends a notification to all users
 			in the room.
 			It uses a default Token to do so!
 
 			.EXAMPLE
 			PS C:\> Send-HipChat -Message "Hello @JoergHochwald" -color "Red" -Room "DevOps" -Token "1234567890" -notify $false
 
 			Description
 			-----------
 			Sent a HipChat Room notification "Hello @JoergHochwald" to the
 			Room "DevOps".
 			The @ indicates a user mention, this is supported like in a regular
 			chat from user 2 User.
 			It uses the Color "red", and it sends no notification to users in
 			the room.
 			It uses a Token "1234567890" to do so! The Token must match the Room!
 
 			.NOTES
 			We use the API V2 now ;-)
 
 			.LINK
 			API: https://www.hipchat.com/docs/apiv2
 
 			.LINK
 			Docs: https://www.hipchat.com/docs/apiv2/method/send_room_notification
 
 			.LINK
 			NET-Experts http://www.net-experts.net
 
 			.LINK
 			Support https://github.com/jhochwald/NETX/issues
 	#>
 
 	[CmdletBinding()]
 	param
 	(
 		[Parameter(HelpMessage = 'HipChat Auth Token')]
 		[Alias('AUTH_TOKEN')]
 		[System.String]$Token = '8EWA77eidxEJG5IFluWjD9794ft8WSzfKhjBCKpv',
 		[Parameter(HelpMessage = 'HipChat Room Name that get the notification')]
 		[Alias('ROOM_ID')]
 		[System.String]$Room = 'Testing',
 		[Parameter(HelpMessage = 'Whether this message should trigger a user notification.')]
 		[boolean]$notify = $false,
 		[Parameter(HelpMessage = 'Background color for message.')]
 		[ValidateSet('yellow', 'green', 'red', 'purple', 'gray', 'random', IgnoreCase = $true)]
 		[System.String]$color = 'gray',
 		[Parameter(HelpMessage = 'The message body')]
 		[ValidateNotNullOrEmpty()]
 		[System.String]$Message,
 		[Parameter(HelpMessage = 'Determines how the message is treated by our server and rendered inside HipChat applications')]
 		[ValidateSet('html', 'text', IgnoreCase = $true)]
 		[Alias('message_format')]
 		[System.String]$Format = 'text'
 	)
 
 	BEGIN {
 		Remove-Variable -Name 'headers' -Force -Confirm:$false -ErrorAction:SilentlyContinue -WarningAction:SilentlyContinue
 		Remove-Variable -Name 'body' -Force -Confirm:$false -ErrorAction:SilentlyContinue -WarningAction:SilentlyContinue
 		Remove-Variable -Name 'myBody' -Force -Confirm:$false -ErrorAction:SilentlyContinue -WarningAction:SilentlyContinue
 		Remove-Variable -Name 'uri' -Force -Confirm:$false -ErrorAction:SilentlyContinue -WarningAction:SilentlyContinue
 		Remove-Variable -Name 'myMethod' -Force -Confirm:$false -ErrorAction:SilentlyContinue -WarningAction:SilentlyContinue
 		Remove-Variable -Name 'post' -Force -Confirm:$false -ErrorAction:SilentlyContinue -WarningAction:SilentlyContinue
 	}
 
 	PROCESS {
 		Set-Variable -Name 'headers' -Value $(@{
 				'Authorization' = "Bearer $($Token)"
 				'Content-type' = 'application/json'
 		})
 
 		$color = $color.ToLower()
 		$Format = $Format.ToLower()
 
 		Set-Variable -Name 'body' -Value $(@{
 				'color'        = "$color"
 				'message_format' = "$Format"
 				'message'      = "$Message"
 				'notify'       = "$notify"
 		})
 
 		Set-Variable -Name 'myBody' -Value $(ConvertTo-Json -InputObject $body -Depth 2 -Compress:$false)
 
 		Set-Variable -Name 'uri' -Value $('https://api.hipchat.com/v2/room/' + $Room + '/notification')
 
 		Set-Variable -Name 'myMethod' -Value $('POST' -as ([System.String] -as [type]))
 
 		try {
 			(Invoke-RestMethod -Uri $uri -Method $myMethod -Headers $headers -Body $myBody -UserAgent "Mozilla/5.0 (Windows NT; Windows NT 6.1; en-US) NET-Experts WindowsPowerShell Service $CoreVersion" -ErrorAction:Stop -WarningAction:SilentlyContinue)
 		} catch [System.Exception] {
 			<#
 					Argh! Catched an Exception...
 			#>
 
 			Write-Error -Message "Error: $($_.Exception.Message) - Line Number: $($_.InvocationInfo.ScriptLineNumber)"
 		} catch {
 			Write-Warning -Message "Could not send notification to your HipChat Room $Room"
 			<#
 					I use Send-HipChat a lot within automated tasks.
 					I post updates from my build server and info from customers Mobile Device Management systems.
 					So I decided to use a warning instead of an error here.
 
 					You might want to change this to fit you needs.
 
 					Remember: If you throw an terminating error, you might want to add a "finally" block to this try/catch Block here.
 			#>
 		} finally {
 			Remove-Variable -Name 'headers' -Force -Confirm:$false -ErrorAction:SilentlyContinue -WarningAction:SilentlyContinue
 			Remove-Variable -Name 'body' -Force -Confirm:$false -ErrorAction:SilentlyContinue -WarningAction:SilentlyContinue
 			Remove-Variable -Name 'myBody' -Force -Confirm:$false -ErrorAction:SilentlyContinue -WarningAction:SilentlyContinue
 			Remove-Variable -Name 'uri' -Force -Confirm:$false -ErrorAction:SilentlyContinue -WarningAction:SilentlyContinue
 			Remove-Variable -Name 'myMethod' -Force -Confirm:$false -ErrorAction:SilentlyContinue -WarningAction:SilentlyContinue
 			Remove-Variable -Name 'post' -Force -Confirm:$false -ErrorAction:SilentlyContinue -WarningAction:SilentlyContinue
 		}
 	}
 }
`

