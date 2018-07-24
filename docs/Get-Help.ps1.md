---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1772
Published Date: 
Archived Date: 2016-10-24t12
---

#  - 

## Description

get-help proxy function with urls for powershell core help topics when using the ‘online’ parameter.

## Comments



## Usage



## TODO



## function

`get-help`

## Code

`#
 #
 function Get-Help
 {
 
 	<#
 	.ForwardHelpTargetName Get-Help
 	.ForwardHelpCategory Cmdlet
 	#>
 
 	[CmdletBinding(DefaultParameterSetName='AllUsersView')]
 	
 	param(
 		[Parameter(Position=0, ValueFromPipelineByPropertyName=$true)]
 		[System.String]
 		${Name},
 
 		[System.String]
 		${Path},
 
 		[System.String[]]
 		${Category},
 
 		[System.String[]]
 		${Component},
 
 		[System.String[]]
 		${Functionality},
 
 		[System.String[]]
 		${Role},
 
 		[Parameter(ParameterSetName='DetailedView')]
 		[Switch]
 		${Detailed},
 
 		[Parameter(ParameterSetName='AllUsersView')]
 		[Switch]
 		${Full},
 
 		[Parameter(ParameterSetName='Examples')]
 		[Switch]
 		${Examples},
 
 		[Parameter(ParameterSetName='Parameters')]
 		[System.String]
 		${Parameter},
 
 		[Switch]
 		${Online}
 	)
 
 	begin
 	{
 		try {
 			$OnlineBaseHelpUrl = 'http://go.microsoft.com/fwlink/?LinkID='
 
 			$about = New-Object PSObject -Property @{
 				'about_Aliases' = 113207
 				'about_Arithmetic_Operators' = 113208
 				'about_Arrays' = 113209
 				'about_Assignment_Operators' = 113210
 				'about_Automatic_Variables' = 113212
 				'about_Break' = 113213
 				'about_Command_Precedence' = 113214
 				'about_Command_Syntax' = 113215
 				'about_Comment_Based_Help' = 144309
 				'about_CommonParameters' = 113216
 				'about_Comparison_Operators' = 113217
 				'about_Continue' = 113218
 				'about_Core_Commands' = 113219
 				'about_Data_Sections' = 113220
 				'about_Debuggers' = 113221
 				'about_Do' = 135169
 				'about_Environment_Variables' = 113222
 				'about_Escape_Characters' = 113223
 				'about_EventLogs' = 113224
 				'about_Execution_Policies' = 135170
 				'about_For' = 113228
 				'about_Foreach' = 113229
 				'about_Format.ps1xml' = 113230
 				'about_Functions' = 113231
 				'about_Functions_Advanced' = 144511
 				'about_Functions_Advanced_Methods' = 135172
 				'about_Functions_Advanced_Parameters' = 135173
 				'about_Functions_CmdletBindingAttribute' = 135174
 				'about_Hash_Tables' = 135175
 				'about_History' = 113233
 				'about_If' = 113234
 				'about_Job_Details' = 135176
 				'about_jobs' = 113251
 				'about_Join' = 113235
 				'about_Language_Keywords' = 136588
 				'about_Line_Editing' = 113236
 				'about_Locations' = 113237
 				'about_Logical_Operators' = 113238
 				'about_Methods' = 113239
 				'about_Modules' = 144311
 				'about_Objects' = 113241
 				'about_Operators' = 113242
 				'about_Parameters' = 113243
 				'about_Parsing' = 113244
 				'about_Path_Syntax' = 113245
 				'about_Pipelines' = 113246
 				'about_Preference_Variables' = 113248
 				'about_Profiles' = 113729
 				'about_Prompts' = 135179
 				'about_Properties' = 113249
 				'about_Providers' = 113250
 				'about_PSSession_Details' = 135180
 				'about_PSSessions' = 135181
 				'about_PSsnapins' = 113252
 				'about_Quoting_Rules' = 113253
 				'about_Redirection' = 113254
 				'about_Ref' = 113255
 				'about_Regular_Expressions' = 113256
 				'about_Remote' = 135182
 				'about_Remote_FAQ' = 135183
 				'about_Remote_Jobs' = 135184
 				'about_Remote_Output' = 135185
 				'about_Remote_Requirements' = 135187
 				'about_Remote_Troubleshooting' = 135188
 				'about_Requires' = 135190
 				'about_Reserved_Words' = 113258
 				'about_Return' = 136587
 				'about_Scopes' = 113260
 				'about_Script_Blocks' = 113261
 				'about_Script_Internationalization' = 113262
 				'about_Scripts' = 144310
 				'about_Session_Configurations' = 145152
 				'about_Signing' = 113268
 				'about_Special_Characters' = 113269
 				'about_Split' = 113270
 				'about_Switch' = 113271
 				'about_Throw' = 145153
 				'about_Transactions' = 135192
 				'about_Trap' = 136586
 				'about_Try_Catch_Finally' = 113444
 				'about_Type_Operators' = 113273
 				'about_Types.ps1xml' = 113274
 				'about_Variables' = 157591
 				'about_While' = 113275
 				'about_Wildcards' = 113276
 				'about_Windows_Powershell_2.0' = 113247
 				'about_Windows_PowerShell_ISE' = 135178
 				'about_WMI_Cmdlets' = 145766
 				'about_WS-Management_Cmdlets' = 145774
 			}
 
 			$outBuffer = $null
 			if ($PSBoundParameters.TryGetValue('OutBuffer', [ref]$outBuffer))
 			{
 				$PSBoundParameters['OutBuffer'] = 1
 			}
 
 			$wrappedCmd = $ExecutionContext.InvokeCommand.GetCommand('Get-Help', [System.Management.Automation.CommandTypes]::Cmdlet)
 
 			if($Name -like 'about_*' -AND $about."$name" -AND $Online)
 			{
 				Start-Process ("$OnlineBaseHelpUrl" + $about."$name")
 				continue
 			}
 
 			$scriptCmd = {& $wrappedCmd @PSBoundParameters }
 			$steppablePipeline = $scriptCmd.GetSteppablePipeline($myInvocation.CommandOrigin)
 			$steppablePipeline.Begin($PSCmdlet)
 
 			} 
 		catch
 		{
 			throw
 		}
 	}
 
 	
 	process
 	{
 		try
 		{        
 			$steppablePipeline.Process($_)
 		}
 		catch 
 		{
 			throw
 		}
 	}
 
 	
 	end
 	{
 		try 
 		{
 			$steppablePipeline.End()
 		}
 		catch 
 		{
 			throw
 		}
 	}
 }
`

