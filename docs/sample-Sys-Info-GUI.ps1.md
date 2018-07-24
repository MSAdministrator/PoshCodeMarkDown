---
Author: briank
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6530
Published Date: 2016-09-26t12
Archived Date: 2016-11-12t03
---

# sample sys info gui - 

## Description

added try block and missing brackets for sample gui from post made by vinith menon.

## Comments



## Usage



## TODO



## function

`load-combobox`

## Code

`#
 #
 $OnLoadFormEvent={
  
 $array = @("Bios_Information","Computer_System_Information","Processor_Information")
  
  
 $array | ForEach-Object {Load-ComboBox -ComboBox $combobox1 -Append -Items $_ }
  
 }
 
 
 
 
 
 function Load-ComboBox 
 {
 <#
 	.SYNOPSIS
 		This functions helps you load items into a ComboBox.
 
 	.DESCRIPTION
 		Use this function to dynamically load items into the ComboBox control.
 
 	.PARAMETER  ComboBox
 		The ComboBox control you want to add items to.
 
 	.PARAMETER  Items
 		The object or objects you wish to load into the ComboBox's Items collection.
 
 	.PARAMETER  DisplayMember
 		Indicates the property to display for the items in this control.
 	
 	.PARAMETER  Append
 		Adds the item(s) to the ComboBox without clearing the Items collection.
 	
 	.EXAMPLE
 		Load-ComboBox $combobox1 "Red", "White", "Blue"
 	
 	.EXAMPLE
 		Load-ComboBox $combobox1 "Red" -Append
 		Load-ComboBox $combobox1 "White" -Append
 		Load-ComboBox $combobox1 "Blue" -Append
 	
 	.EXAMPLE
 		Load-ComboBox $combobox1 (Get-Process) "ProcessName"
 #>
 	Param (
 		[ValidateNotNull()]
 		[Parameter(Mandatory=$true)]
 		[System.Windows.Forms.ComboBox]$ComboBox,
 		[ValidateNotNull()]
 		[Parameter(Mandatory=$true)]
 		$Items,
 	    [Parameter(Mandatory=$false)]
 		[string]$DisplayMember,
 		[switch]$Append
 	)
 	
 	if(-not $Append)
 	{
 		$ComboBox.Items.Clear()	
 	}
 	
 	if($Items -is [Object[]])
 	{
 		$ComboBox.Items.AddRange($Items)
 	}
 	elseif ($Items -is [Array])
 	{
 		$ComboBox.BeginUpdate()
 		foreach($obj in $Items)
 		{
 			$ComboBox.Items.Add($obj)	
 		}
 		$ComboBox.EndUpdate()
 	}
 	else
 	{
 		$ComboBox.Items.Add($Items)	
 	}
 
 	$ComboBox.DisplayMember = $DisplayMember	
 }
 
 $buttonResetComputerName_Click={
 	$textbox1.Clear()
 }
 
 $buttonGO_Click={
 Try {
  
 	if ($textbox1.Text -ne $null)
 	{
 	 
 	 if ($combobox1.SelectedIndex -gt -1 -and $combobox1.SelectedItem -eq "Bios_Information")
 	 
 	 {
 	 
 	$servername = $textbox1.Text
 	 
 	 Get-WmiObject -Class win32_bios -ComputerName $servername -ea 'Stop' |
 	 
 	 Out-GridView -Title "$($combobox1.SelectedItem) for $servername"
 	 
 	}
 	 
 	 
 	 elseif ($combobox1.SelectedIndex -gt -1 -and $combobox1.SelectedItem -eq "Computer_System_Information")
 	 
 	 {
 	 
 	$servername = $textbox1.Text
 	 
 	Get-WmiObject -Class Win32_ComputerSystem -ComputerName $servername -ea 'Stop' |
 	 
 	 Out-GridView -Title "$($combobox1.SelectedItem) for $servername"
 	 
 	 }
 	 
 	 
 	 so that we can trap the error in try catch block #>
 	 
 	 elseif ($combobox1.SelectedIndex -gt -1 -and $combobox1.SelectedItem -eq "Processor_Information")
 	 
 	 {
 	 
 	 
 	$servername = $textbox1.Text
 	 
 	 Get-WmiObject -Class Win32_Processor -ComputerName $servername -ea 'Stop' |
 	 
 	 Out-GridView -Title "$($combobox1.SelectedItem) for $servername"
 	 
 	 }
 	 
 	 }
 	 
 	}
 
  
  catch {
  [void][System.Windows.Forms.MessageBox]::Show(" $($servername) is ShutDown or not Reachable over the Network","Information")
  
 }
 }
`

