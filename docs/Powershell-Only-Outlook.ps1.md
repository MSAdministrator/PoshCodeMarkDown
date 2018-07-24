---
Author: support
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 6215
Published Date: 2016-02-12t08
Archived Date: 2016-03-19t00
---

# powershell only outlook - 

## Description

my first contribution.  i am crazily thinking of doing a “powershell only” day.  the first task is to figure out how to manipulate outlook through powershell.  the submitted script hits my outlook inbox and goes through the inbox and each subfolder and retrieves the unread emails from it.  it then goes through my task list and gets all the incomplete tasks.  this was my first time using a status bar and definitely the first for making anything outside the scripting games public.  i’d hate to get finished with the outlook “module” and find out i could have saved myself a lot of time, so i through the script as it is now on the mercy of the court.  proceed with your red pens…

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $GetOutlook = New-Object -com "Outlook.Application"; 
 $olName = $GetOutlook.GetNamespace("MAPI")
 $olxEmailFolder = $olName.GetDefaultFolder(�olFolderInbox�)
 $olxEmailFolder.Name
 $olxEmailItem = $olxemailFolder.items
 $olxEmailItem | ?{$_.Unread -eq $True} | select SentOn, SenderName, Subject, Body | Format-Table -auto
 $SubFolders = $olxEmailFolder.Folders
 ForEach($Folder in $SubFolders)
 {
 	$Folder.Name
 	$SubfolderItem = $Folder.Items
 	$EmailCount = 1
 	ForEach($Email in $SubfolderItem)
 	{
 		Do
 		{
 			Write-Progress -Activity "Checking folder" -status $Folder.Name -
 
 PercentComplete ($EmailCount/$Folder.Items.Count*100)
 			$EmailCount++
 		}
 		While($EmailCount -le $Folders.Item.Count)
 	$Email | ?{$_.Unread -eq $True} | select SentOn, SenderName, Subject, Body | Format-Table -
 
 Auto
 	}
 }
 $olxTasksFolder = $olName.GetDefaultFolder(�olFolderTasks�)
 $olxTaskItems = $olxTasksFolder.items
 $TaskCount = 1
 $TaskList = @()
 ForEach($TaskItem in $olxTaskItems)
 {
 	Do
 	{
 		Write-Progress -Activity "Checking" -status "Tasks" -PercentComplete ($Taskcount/
 
 $olxTasksFolder.Items.count*100)
 		$TaskCount++
 	}
 	While($TaskCount -le $olxTaskFolder.Items.Count)
 	If($TaskItem.Complete -eq $False)
 	{
 	$TaskList+=New-Object -TypeName PSObject -Property @{
 	Subject=$TaskItem.Subject
 	DateCreated=$TaskItem.CreationTime
 	Percent=$TaskItem.PercentComplete
 	Due=$TaskItem.DueDate
 	}}
 }
 $TaskList | Sort DueDate -descending | Format-Table -Auto
`

