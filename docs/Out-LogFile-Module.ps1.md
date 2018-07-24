---
Author: mike ling
Publisher: 
Copyright: 
Email: 
Version: 2.1
Encoding: ascii
License: cc0
PoshCode ID: 3232
Published Date: 2012-02-12t11
Archived Date: 2012-02-15t00
---

# out-logfile module - 

## Description

log file managment advanced function/ module. supports writehost, pipleline, encoding, blanklines, datestring formatting and others.

## Comments



## Usage



## TODO



## function

`out-logfile`

## Code

`#
 #
 function Out-LogFile {
 <#
 	.SYNOPSIS
 		Output to log file.
 	
 	.DESCRIPTION
 		Output to formatted log file.
 		
 		Default fomatting is:
 		SEVERITY: dd-MM-yyyy HH:mm:ss: Message
 		e.g. I: 25-06-2010 17:09:10: Started Processing
 		
 	.PARAMETER BackgroundColor
 		Specifies the background colour when using -WriteHost. There is no default.
 		
 	.PARAMETER BlankLine
 		Write 1 or more blank lines to the log file (default 0).
 		When used with -Message the blank lines are written after the message.
 		
 	.PARAMETER DateFormat
 		  Date formating string (default "dd-MM-yyyy HH:mm:ss").
 		  See Get-Date -Format
 		
 	.PARAMETER DontFormat
 		Don't format the message and write the input directly to the log file.
 	
 	.PARAMETER Encoding
 		Specifies the type of character encoding used in the file (default Unicode).				
 		Valid values are "Unicode", "UTF7", "UTF8", "UTF32", "ASCII", "BigEndianUnicode", "Default", and "OEM". 
 		"Default" uses the encoding of the system's current ANSI code page.
 		"OEM" uses the current original equipment manufacturer code page identifier for the operating system.
 		
 	.PARAMETER Force
 		Overwrite an existing read-only file.		
 		
 	.PARAMETER ForegroundColor
 		Specifies the text colour when using -WriteHost. There is no default.
 	
 	.PARAMETER LogFile
 		Path to the log file (default $global:DefaultLogFile).
 		Set the global variable $global:DefaultLogFile to use the default setting.
 	
 	.PARAMETER Message
 		Input object or text message to write to the log file.
 		
 	.PARAMETER Severity
 		Message severity flag (default I).
 		Any string can be used as a flag such as I, W, E or INFO, WARNING, ERROR etc.
 		
 	.PARAMETER Title
 		Write unformatted string to the log file with blank lines either side.
 		This is the equivilent of:
 		Out-LogFile -BlankLine 1
 		Out-LogFile -Message "Message" -BlankLine 1 -DontFormat
 
 		-Title is processed before -Message so the two can be combined in a single call:
 		Out-LogFile -Title "Today's Report" -Message "Started Processing"
 	
 	.PARAMETER WriteHost
 		Writes -Message or -Title to the host (default $global:WriteHostPreference).
 		Set the global variable $global:WriteHostPreference to $true to enable -WriteHost by default.
 		To override the default set -WriteHost or -WriteHost:$false.
 		
 	.EXAMPLE	
 		$global:WriteHostPreference = $true
 		$global:DefaultLogFile = "C:\Logs\MyLog.LOG"
 		Out-LogFile "Simple Example"
 		I: 25-06-2010 18:07:00: Simple Example
 		
 		Write "Simple Example" to the default log specified by $global:DefaultLogPath and the host.
 		
 	.EXAMPLE			
 		Out-LogFile "Another Example" -WriteHost -ForgroundColor Green -LogFile C:\Logs\MyLog.LOG -Severity E
 		E: 25-06-2010 18:10:00: Another Example
 		
 		Write "Another Example" to C:\Logs\MyLog.LOG and to the host with green text setting the severity flag to 'E'.
 	
 	.EXAMPLE	
 		Daily-Report | Out-LogFile -Title "Todays Report" -DontFormat -BlankLine 1
 				
 		Write the log title "Daily Report" followed by the output from Daily-Report to the default log file without formatting. A single blank line is added at the end.
 		
 	.INPUTS
 		Object
 		
 	.OUTPUTS
 		None
 	
 	.NOTES
 		Author: Mike Ling
 		Histroy:
 		v2.1 - 13/02/2012 - Added support for: $global:DefaultLogFile and $global:WriteHostPreference
 		v2.0 - 07/02/2012 - Added support for: Pipleline, CmdletBinding, ParameterSets, Encoding, WriteHost Colour.
 		v1.0 - 25/06/2010 - First release.		
 	
 	.LINK
 	Out-File
 #>
 	[CmdletBinding(DefaultParameterSetName='Message')]
 	param(
 		[Parameter(ParameterSetName='Message',
 		Position=0,
 		ValueFromPipeline=$true)]				
 		[object[]]$Message,
 		[Parameter(ParameterSetName='Message')]
 		[string]$LogFile = $global:DefaultLogPath,
 		[Parameter(ParameterSetName='Message')]
 		[int]$BlankLine = 0,		
 		[switch]$WriteHost = $global:WriteHostPreference,
 		[string]$Severity = "I",
 		[Parameter(ParameterSetName='Message')]
 		[switch]$DontFormat,		
 		[Parameter(ParameterSetName='Message')]
 		[string]$DateFormat = "dd-MM-yyyy HH:mm:ss",		
 		#[Parameter(ParameterSetName='Title',Position=0,Mandatory=$true)]
 		[string]$Title,		
 		[System.ConsoleColor]$ForegroundColor = $host.UI.RawUI.ForegroundColor,		
 		[System.ConsoleColor]$BackgroundColor = $host.UI.RawUI.BackgroundColor,
 		[ValidateSet('unicode', 'utf7', 'utf8', 'utf32', 'ascii', 'bigendianunicode', 'default', 'oem')]		
 		[string]$Encoding = 'Unicode',
 		[switch]$Force
 	)
 	
 	begin { 		
 		Write-Verbose "Log File: $LogFile"
 		if ( -not $LogFile ) { Write-Error "The -LogFile parameter must be defined or $global:LogFile must be set."; break}		
 		if ( -not (Test-Path $LogFile) ) { New-Item -Path $LogFile -ItemType File -Force | Out-Null }
 		if ( -not (Test-Path $LogFile) ) { Write-Error "Log file can not be found: $LogFile."; break}
 		
 		if ( $Title ) {			
 			$text = $Title
 			$Title = $null
 			Out-LogFile -BlankLine 1 -LogFile $LogFile -WriteHost:$WriteHost -Force:$Force -Encoding $Encoding
 			Out-LogFile -Message $text -BlankLine 1 -DontFormat -LogFile $LogFile -WriteHost:$WriteHost -Force:$Force -Encoding $Encoding 									
 		}
 	}
 	process {
 		
 		if ( $Message ) { 	
 			$text = $Message
 			foreach ( $text in $Message ) {
 				if ( -not $DontFormat ) { $text = "$(($Severity).ToUpper()): $(Get-Date -Format `"$DateFormat`")" + ": $text" }									
 				if ($WriteHost) { Write-Host $text -BackgroundColor $BackgroundColor -ForegroundColor $ForegroundColor}
 				$text | Out-File -FilePath $LogFile -Force:$Force -Encoding $Encoding -Append
 			}		
 		}
 		
 		if ( $BlankLine -gt 0 ){
 			for ($i = 0; $i -lt $BlankLine; $i++ ) { 
 				"" | Out-File -FilePath $LogFile -Force:$Force -Encoding $Encoding -Append
 				if ($WriteHost) { Write-Host "" -BackgroundColor $BackgroundColor -ForegroundColor $ForegroundColor }
 			}
 		}
 	}
 	end {
 	}
 }
`

