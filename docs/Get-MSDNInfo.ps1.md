---
Author: shay levy
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2278
Published Date: 2012-10-04t05
Archived Date: 2012-12-29t04
---

# get-msdninfo - 

## Description

opens the msdn web page of an object member

## Comments



## Usage



## TODO



## function

`get-msdninfo`

## Code

`#
 #
 
 function Get-MSDNInfo
 {
 
 	<#
 	.SYNOPSIS
 		Opens the MSDN web page of an object member: type, method or property. 
 
 	.DESCRIPTION
        		The Get-MSDNInfo function enables you to quickly open a web browser to the MSDN web
        		page of any given instance of a .NET object. You can also refer to Get-MSDNInfo by its alias: 'msdn'.     
         
 	.PARAMETER InputObject
 		   Specifies an instance of a .NET object. 
 
 	.PARAMETER Name 	
 		Specifies one member name (type, property or method name).
   
 	.PARAMETER MemberType
 		Specifies the type of the object member. 
 		Possible values are: 'Type','Property','Method' or their shortcuts: 'T','M','P'. 
 		The default value is 'Type'.
 	
 	.PARAMETER Culture
 		If MSDN has localized versions then you can use this parameter with a value of the localized 
 		version culture string. The default value is 'en-US'. The full list of cultures can be found by 
 		executing the following command:
 
 		PS > [System.Globalization.CultureInfo]::GetCultures('AllCultures')
 
 	.PARAMETER List
 		The List parameter orders the function to print a list of the incoming object's methods and properties.
 		You can use the List parameter to get a view of the object members that the function support.
 
 	.EXAMPLE
 		PS> $ps = Get-Process -Id $pid
 		PS> $ps | Get-MSDNInfo -List
 
 			 TypeName: System.Diagnostics.Process
 
 		MemberType Name
 		---------- ----
 		    Method BeginErrorReadLine
 		    Method BeginOutputReadLine
 		    Method CancelErrorRead
 		    Method CancelOutputRead
 		    Method Close
 		    Method CloseMainWindow
 		    Method CreateObjRef
 		    Method Dispose
 		    Method EnterDebugMode
 		    Method Equals
 		    Method GetCurrentProcess
 		    Method GetHashCode
 		    Method GetLifetimeService
 		    Method GetProcessById
 		    Method GetProcesses
 		    (...)
 		  Property BasePriority
 		  Property Container
 		  Property EnableRaisingEvents
 		  Property ExitCode
 		  Property ExitTime
 		  Property Handle
 		  Property HandleCount
 		  Property HasExited
 		  Property Id
 		  Property MachineName
 		  Property MainModule
 		  (...)
 
 
 		Description
 		-----------	
 		The first command store the current session process object in a variable named $ps.
 		The variable is piped to Get-MSDNInfo which specifies the List parameter. The result is a list of the process object members.
 
 	.EXAMPLE
 		PS> $date = Get-Date
 		PS> Get-MSDNInfo -InputObject $date -MemberType Property
 
 		  TypeName: System.DateTime
 
 		1. Date
 		2. Day
 		3. DayOfWeek
 		4. DayOfYear
 		5. Hour
 		6. Kind
 		7. Millisecond
 		8. Minute
 		9. Month
 		10. Now
 		11. Second
 		12. Ticks
 		13. TimeOfDay
 		14. Today
 		15. UtcNow
 		16. Year
 
 		Enter the Property item number ( Between 1 and 16. Press CTRL+C to cancel):
 
 
 		Description
 		-----------
 		The command gets all the properties of the date object and prints a numbered menu. 
 		You are asked to supply the property item number. Once you type the number and press the Enter key, the function 
 		invokes the given property web page in the default browser.		
 
 	.EXAMPLE
 		
 		PS> $date = Get-Date
 		PS> Get-MSDNInfo -InputObject $date
 		
 		Description
 		-----------	
 		Opens the web page of the object type name.	
 
 	.EXAMPLE
 		Get-Date | Get-MSDNInfo -Name DayOfWeek -MemberType Property
 		
 		Description
 		-----------		
 		If you know the exact property name and you don't want the function to print the numbered menu then 
 		you can pass the property name as an argument to the MemberType parameter. This will open the MSDN 
 		web page of the DayOfWeek property of the DateTime structure. 
 		
 	.EXAMPLE
 				
 		PS> $winrm = Get-Service -Name WinRM
 		PS> $winrm | msdn -mt Method
 		
 		  TypeName: System.ServiceProcess.ServiceController
 
 		1. Close
 		2. Continue
 		3. CreateObjRef
 		4. Dispose
 		5. Equals
 		6. ExecuteCommand
 		7. GetDevices
 		8. GetHashCode
 		9. GetLifetimeService
 		10. GetServices
 		11. GetType
 		12. InitializeLifetimeService
 		13. Pause
 		14. Refresh
 		15. Start
 		16. Stop
 		17. ToString
 		18. WaitForStatus
 
 		Enter the Method item number ( Between 1 and 18. Press CTRL+C to cancel):
 
 
 		Description
 		-----------
 		The command prints a numbered menu of the WinRM service object methods.
 		You are asked to supply the method item number. Once you type the number and press the Enter
 		key, the function invokes the given method web page in the default browser.
 		The command also uses the 'msdn' alias of the function Get-MSDNInfo and the alias 'mt' of the MemberType parameter.
 		
 	.EXAMPLE
 
 		$date | Get-MSDNInfo -Name AddDays -MemberType Method
 		
 		Description
 		-----------		
 		Opens the page of the AddDays DateTime method.
 			
 	.NOTES
 		Author: Shay Levy
 		Blog  : http://blogs.microsoft.co.il/blogs/ScriptFanatic/
 
 	.LINK
 		 http://blogs.microsoft.co.il/blogs/ScriptFanatic/
 	#>
 	
 	
 	[CmdletBinding(DefaultParameterSetName='Type')]
 	
 	param(
 		[Parameter(Mandatory=$true,Position=0,ValueFromPipeline=$true)]
 		[ValidateNotNullOrEmpty()]
 		[System.Object]$InputObject,
 
 		[Parameter(ParameterSetName='Type')]
 		[ValidateNotNullOrEmpty()]
 		[System.String]$Name,
 		
 		[Parameter(ParameterSetName='Type')]
 		[ValidateNotNullOrEmpty()]
 		[ValidateSet('Type','Method','Property','t','m','p')]
 		[Alias('mt')]
 		[System.String]$MemberType='Type',
 
 		[Parameter(ParameterSetName='Type')]
 		[ValidateNotNullOrEmpty()]
 		[System.String]$Culture='en-US',
 		
 		[Parameter(ParameterSetName='List')]
 		[switch]$List
 	)
 
 	begin
 	{		
 		$baseUrl = "http://msdn.microsoft.com/$Culture/library/"
 		
 		$SpecialName = [System.Reflection.MethodAttributes]::SpecialName
 		
 		switch -regex($MemberType)
 		{
 			'^(Type|t)$' {$MemberType='Type'; break}
 			'^(Method|m)$' {$MemberType='Method'; break}
 			'^(Property|p)$' {$MemberType='Property'; break}
 			default {$MemberType='Type'}
 		}
 	}
 	
 	process
 	{		
 		if($MemberType -eq 'Type' -AND $Name)
 		{
 			Throw "Invalid 'MemberType' value. Please enter another value: 'Property' or 'Method'."
 		}
 		
 		if($PSCmdlet.ParameterSetName -eq 'List')
 		{			
 			Write-Host "`n`t TypeName: $($InputObject.GetType().FullName)"
 			$InputObject.GetType().GetMembers() | Where-Object {$_.MemberType -match '^(method|property)$' -AND !($_.Attributes -band $SpecialName)} | Sort-Object MemberType,Name -Unique | Format-Table MemberType,Name -Auto
 		}
 		else
 		{
 			$member = $InputObject.GetType().GetMembers() | Where-Object {$_.MemberType -eq $MemberType -AND !($_.Attributes -band $SpecialName)} | Select-Object Name,MemberType,DeclaringType -Unique | Sort-Object Name
 
 			if($MemberType -eq 'Type')
 			{
 				Start-Process -FilePath ("$baseUrl{0}.aspx" -f $InputObject.GetType().FullName)
 				break
 			}
 
 			if($Name)
 			{
 				$m = $member | Where-Object {$_.MemberType -eq $MemberType -AND $_.Name -eq $Name}
 
 				if($m)
 				{
 					Start-Process -FilePath ("$baseUrl{0}.{1}.aspx" -f $m.DeclaringType,$m.Name)
 				}
 				else
 				{
 					Write-Error "'$Name' is not a $MemberType of '$($InputObject.GetType().FullName)'. Check the Name and try again."
 				}
 			}
 			else
 			{
 				Write-Host "`n  TypeName: $($InputObject.GetType().FullName)`n"
 
 				for($i=0;$i -lt $member.count; $i++)
 				{
 					Write-Host "$($i+1). $($member[$i].name)"
 				}
 
 				do{
 					[int]$num = Read-Host -Prompt "`nEnter the $membertype item number ( Between 1 and $($member.count). Press CTRL+C to cancel)"
 				} while ( $num -lt 1 -or $num -gt $member.count)
 
 				Start-Process -FilePath ("$baseUrl{0}.{1}.aspx" -f $member[$num-1].DeclaringType,$member[$num-1].Name)
 			}
 		}
 	}
 }
 
 Set-Alias -Name msdn -Value Get-MSDNInfo
`

