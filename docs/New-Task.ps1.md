---
Author: mark e
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 829
Published Date: 
Archived Date: 2009-10-25t10
---

# new-task - 

## Description

allows for the creation of tasks in microsoft outlook from windows powershell. the majority of task options available can be configured with the function.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 .SYNOPSIS
 Creates a task in Outlook
 .DESCRIPTION
 Allows for the creation of tasks in Microsoft Outlook from Windows PowerShell. The majority of task options available can be configured with the function.
 .LINK
 http://www.leeholmes.com/blog/GettingThingsDoneOutlookTaskAutomationWithPowerShell.aspx
 .EXAMPLE
 	C:\PS>New-Task -Subject "Create PowerShell script to create Outlook tasks"
 	
 	Description
 	-----------
 	This example just a simple Outlook Task with the specified subject.
  .EXAMPLE
   C:\PS>New-Task -Subject "Create powershell script to create outlook calendar entries" -Categories "%PowerShell" -DueDate "2/01/2009" -Status "InProgress" -Display
   
   Description
   -----------
   This example demonstrates the creation of task using additional options. The '-display' parameter brings up the Outlook task form at the end to allow you make additional changes in the UI.
 .INPUTTYPE
 .RETURNVALUE
 .COMPONENT
 .ROLE
 .FUNCTIONALITY
 .PARAMETER Subject
   Sets the subject of the task.
 .PARAMETER Body
   Sets the body of the task.
 .PARAMETER Categories
   Sets the categories for the task. Comma seperate the values for multiple categories.
 .PARAMETER StartDate
   Sets the start date of the task. Must be in a format that can be converted to [datetime] object.
 .PARAMETER DueDate
   Sets the due date of the task. Must be in a format that can be converted to [datetime] object.
 .PARAMETER PercentComplete
   Sets the percent complete for the new task. Value must be an int from 0 to 100.
 .PARAMETER ReminderTime
   Sets the reminder time of the task. Must be in a format that can be converted to [datetime] object.
 .PARAMETER Status
   Sets the the status of the new task. Valid values are "Completed", "Deferred", "InProgress","Waiting"
 .PARAMETER Priority
   Sets the priority value of the new task. Valid values are "Low", "Medium", "High"
 .PARAMETER Display
 	Displays the Outlook task form after saving.
 .NOTES
 Name: New-Task
 Author: Mark E. Schill
 Date Created: 01/24/2009
 Date Revised: 01/24/2009
 Version: 1.0
 History: 1.0 01/24/2009 - Initial Revision
 #>
 [CmdletBinding()]
 param(
 [Parameter(Position=0, Mandatory=$true, ValueFromPipeline=$true)]
 [string]
 $Subject
 ,
 [Parameter(Mandatory=$false, ValueFromPipeline=$false)]
 [string]
 $Body
 ,
 [Parameter(Mandatory=$false, ValueFromPipeline=$false)]
 [DateTime]
 $StartDate
 ,
 [Parameter(Mandatory=$false, ValueFromPipeline=$false)]
 [DateTime]
 $DueDate
 ,
 [Parameter(Mandatory=$false, ValueFromPipeline=$false)]
 [string]
 $Status
 ,
 [Parameter(Mandatory=$false, ValueFromPipeline=$false)]
 [string]
 $Priority
 ,
 [Parameter(Mandatory=$false, ValueFromPipeline=$false)]
 [int]
 $PercentComplete = 0
 ,
 [Parameter(Mandatory=$false, ValueFromPipeline=$false)]
 [DateTime]
 $ReminderTime
 ,
 [Parameter(Mandatory=$false, ValueFromPipeline=$false)]
 [string]
 $Categories
 ,
 [Parameter(Mandatory=$false, ValueFromPipeline=$false)]
 [switch]
 $Display
 )
 Begin
 {
 	$Outlook = New-Object -ComObject Outlook.Application
 }
 Process
 {
 	$Task = $Outlook.Application.CreateItem($olTaskItem)
 	$ContainsError = $false
 
 	$Task.Subject = $Subject
 	$Task.PercentComplete = $PercentComplete
 	$Task.Body = $Body
 	if ( $Categories )  { $Task.Categories = $Categories }  
  	if ( $StartDate ) { $Task.StartDate = $StartDate }
   if ( $DueDate ) { $Task.DueDate   = $DueDate }
 	 
   switch ( $Status )
     {
       "Completed" 
         { 
           $Task.Status = [Microsoft.Office.Interop.Outlook.OlTaskStatus]::olTaskComplete 
         }
        "Deferred" { $Task.Status = [Microsoft.Office.Interop.Outlook.OlTaskStatus]::olTaskDeferred }
        "InProgress" { $Task.Status = [Microsoft.Office.Interop.Outlook.OlTaskStatus]::olTaskInProgress }
        "Waiting" { $Task.Status = [Microsoft.Office.Interop.Outlook.OlTaskStatus]::olTaskWaiting }
        default { $Task.Status = [Microsoft.Office.Interop.Outlook.OlTaskStatus]::TaskNotStarted }
     }
   
   switch ($Priority )
     {
        "Low"  { $Task.Importance = [Microsoft.Office.Interop.Outlook.OlImportance]::olImportanceLow }
        "High" { $Task.Importance = [Microsoft.Office.Interop.Outlook.OlImportance]::olImportanceHigh }
        default { $Task.Importance = [Microsoft.Office.Interop.Outlook.OlImportance]::olImportanceNormal }
     }
   
   
 	if ( $ReminderTime )
 		{
 		$Task.ReminderTime = $ReminderTime
 		$Task.ReminderSet = $True
 		}
   	
   if ( -not $ContainsError)
     { 
 		$Task.Save()
 		if ($Display ) { $Task.Display() } 
     }
   
 }
 End
 {
   $Outlook = $Null
 }
`

