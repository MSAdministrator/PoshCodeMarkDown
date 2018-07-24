---
Author: joerg hochwald
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 6431
Published Date: 2016-07-01t04
Archived Date: 2016-08-09t21
---

# approve-mailaddress - 

## Description

checks a given mail address against a regex filter to see if it is rfc822 complaint

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
 
 
 function global:Approve-MailAddress {
 	<#
 			.SYNOPSIS
 			REGEX check to see if a given Email address is valid
 
 			.DESCRIPTION
 			Checks a given Mail Address against a REGEX Filter to see if it is
 			RfC822 complaint
 			Not directly related is the REGEX check. Most mailer will not be able
 			to handle it if there are non standard chars within the Mail Address...
 
 			.PARAMETER Email
 			e.g. "joerg.hochwald@outlook.com"
 			Email address to check
 
 			.EXAMPLE
 			PS C:\> Approve-MailAddress -Email:"No.Reply@bewoelkt.net"
 			True
 
 			Description
 			-----------
 			Checks a given Mail Address (No.Reply@bewoelkt.net) against a REGEX
 			Filter to see if it is RfC822 complaint
 
 			.EXAMPLE
 			PS C:\> Approve-MailAddress -Email:"Jörg.hochwald@gmail.com"
 			False
 
 			Description
 			-----------
 			Checks a given Mail Address (JÃ¶rg.hochwald@gmail.com) against a
 			REGEX Filter to see if it is RfC822 complaint, and it is NOT
 
 			.EXAMPLE
 			PS C:\> Approve-MailAddress -Email:"Joerg hochwald@gmail.com"
 			False
 
 			Description
 			-----------
 			Checks a given Mail Address (Joerg hochwald@gmail.com) against a
 			REGEX Filter to see if it is RfC822 complaint, and it is NOT
 
 			.EXAMPLE
 			PS C:\> Approve-MailAddress -Email:"Joerg.hochwald@gmail"
 			False
 
 			Description
 			-----------
 			Checks a given Mail Address (Joerg.hochwald@gmail) against a
 			REGEX Filter to see if it is RfC822 complaint, and it is NOT
 
 			.NOTES
 			Internal Helper function to check Mail addresses via REGEX to see
 			if they are RfC822 complaint before use them.
 
 			.LINK
 			NET-Experts http://www.net-experts.net
 
 			.LINK
 			Support https://github.com/jhochwald/NETX/issues
 	#>
 
 	[CmdletBinding()]
 	[OutputType([bool])]
 	param
 	(
 		[Parameter(Mandatory = $true,
 		HelpMessage = 'Enter the Mail Address that you would like to check (Mandatory)')]
 		[ValidateNotNullOrEmpty()]
 		[Alias('Mail')]
 		[System.String]$Email
 	)
 
 	BEGIN {
 		Set-Variable -Name 'EmailRegexOld' -Value $("^(?("")("".+?""@)|(([0-9a-zA-Z]((\.(?!\.))|[-!#\$%&'\*\+/=\?\^`\{\}\|~\w])*)(?<=[0-9a-zA-Z])@))(?(\[)(\[(\d{1,3}\.){3}\d{1,3}\])|(([0-9a-zA-Z][-\w]*[0-9a-zA-Z]\.)+[a-zA-Z]{2,6}))$" -as ([regex] -as [type]))
 
 		Set-Variable -Name 'EmailRegex' -Value $('^[_a-zA-Z0-9-]+(\.[_a-zA-Z0-9-]+)*@[a-zA-Z0-9-]+(\.[a-zA-Z0-9-]+)*(\.[a-zA-Z]{2,6})$' -as ([regex] -as [type]))
 	}
 
 	PROCESS {
 		if (($Email -match $EmailRegexOld) -and ($Email -match $EmailRegex)) {
 			Return $true
 		} else {
 			Return $false
 		}
 	}
 }
`

