---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 3.1.15
Encoding: ascii
License: cc0
PoshCode ID: 3990
Published Date: 
Archived Date: 2013-03-02t17
---

#  - 

## Description

provides a gui console for creating single or multiple folders in the configuration manager 2012 sp1 console using a csv import. useful in automating bulk folder creations within the config manager 2012 sp1 console.

## Comments



## Usage



## TODO



## function

`call-createfolder_pff`

## Code

`#
 #
 #========================================================================
 #========================================================================
 #----------------------------------------------
 #----------------------------------------------
 
 function OnApplicationLoad {
 	
 }
 
 function OnApplicationExit {
 	
 }
 
 
 #----------------------------------------------
 #----------------------------------------------
 function Call-CreateFolder_pff {
 
 	#----------------------------------------------
 	#----------------------------------------------
 	[void][reflection.assembly]::Load("mscorlib, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089")
 	[void][reflection.assembly]::Load("System, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089")
 	[void][reflection.assembly]::Load("System.Windows.Forms, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089")
 	[void][reflection.assembly]::Load("System.Data, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089")
 	[void][reflection.assembly]::Load("System.Drawing, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a")
 	[void][reflection.assembly]::Load("System.Xml, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089")
 	[void][reflection.assembly]::Load("System.DirectoryServices, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a")
 	[void][reflection.assembly]::Load("System.Core, Version=3.5.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089")
 	[void][reflection.assembly]::Load("System.ServiceProcess, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a")
 
 	#----------------------------------------------
 	#----------------------------------------------
 	try{
 		$local:type = [ProgressBarOverlay]
 	}
 	catch
 	{
 		Add-Type -ReferencedAssemblies ('System.Windows.Forms', 'System.Drawing') -TypeDefinition  @" 
 		using System;
 		using System.Windows.Forms;
 		using System.Drawing;
         namespace SAPIENTypes
         {
 		    public class ProgressBarOverlay : System.Windows.Forms.ProgressBar
 	        {
 	            protected override void WndProc(ref Message m)
 	            { 
 	                base.WndProc(ref m);
 	                if (m.Msg == 0x000F)// WM_PAINT
 	                {
 	                    if (Style != System.Windows.Forms.ProgressBarStyle.Marquee || !string.IsNullOrEmpty(this.Text))
                         {
                             using (Graphics g = this.CreateGraphics())
                             {
                                 using (StringFormat stringFormat = new StringFormat(StringFormatFlags.NoWrap))
                                 {
                                     stringFormat.Alignment = StringAlignment.Center;
                                     stringFormat.LineAlignment = StringAlignment.Center;
                                     if (!string.IsNullOrEmpty(this.Text))
                                         g.DrawString(this.Text, this.Font, Brushes.Black, this.ClientRectangle, stringFormat);
                                     else
                                     {
                                         int percent = (int)(((double)Value / (double)Maximum) * 100);
                                         g.DrawString(percent.ToString() + "%", this.Font, Brushes.Black, this.ClientRectangle, stringFormat);
                                     }
                                 }
                             }
                         }
 	                }
 	            }
               
                 public string TextOverlay
                 {
                     get
                     {
                         return base.Text;
                     }
                     set
                     {
                         base.Text = value;
                     }
                 }
 	        }
         }
 "@ | Out-Null
 	}
 
 	#----------------------------------------------
 	#----------------------------------------------
 	[System.Windows.Forms.Application]::EnableVisualStyles()
 	$formConsoleFolderCreator = New-Object 'System.Windows.Forms.Form'
 	$progressbaroverlay1 = New-Object 'SAPIENTypes.ProgressBarOverlay'
 	$button1Create = New-Object 'System.Windows.Forms.Button'
 	$datagridview2 = New-Object 'System.Windows.Forms.DataGridView'
 	$buttonBrowse = New-Object 'System.Windows.Forms.Button'
 	$textbox2 = New-Object 'System.Windows.Forms.TextBox'
 	$labelCSVFile = New-Object 'System.Windows.Forms.Label'
 	$labelProgress = New-Object 'System.Windows.Forms.Label'
 	$datagridview1 = New-Object 'System.Windows.Forms.DataGridView'
 	$buttonCreate = New-Object 'System.Windows.Forms.Button'
 	$combobox1 = New-Object 'System.Windows.Forms.ComboBox'
 	$textbox1 = New-Object 'System.Windows.Forms.TextBox'
 	$labelName = New-Object 'System.Windows.Forms.Label'
 	$openfiledialog1 = New-Object 'System.Windows.Forms.OpenFileDialog'
 	$InitialFormWindowState = New-Object 'System.Windows.Forms.FormWindowState'
 
 	#----------------------------------------------
 	#----------------------------------------------
 	
 	
 	
 	
 	
 	
 	
 	$formConsoleFolderCreator_Load={
 		
 	}
 	
 	function Load-DataGridView
 	{
 		<#
 		.SYNOPSIS
 			This functions helps you load items into a DataGridView.
 	
 		.DESCRIPTION
 			Use this function to dynamically load items into the DataGridView control.
 	
 		.PARAMETER  DataGridView
 			The ComboBox control you want to add items to.
 	
 		.PARAMETER  Item
 			The object or objects you wish to load into the ComboBox's items collection.
 		
 		.PARAMETER  DataMember
 			Sets the name of the list or table in the data source for which the DataGridView is displaying data.
 	
 		#>
 		Param (
 			[ValidateNotNull()]
 			[Parameter(Mandatory=$true)]
 			[System.Windows.Forms.DataGridView]$DataGridView,
 			[ValidateNotNull()]
 			[Parameter(Mandatory=$true)]
 			$Item,
 		    [Parameter(Mandatory=$false)]
 			[string]$DataMember
 		)
 		$DataGridView.SuspendLayout()
 		$DataGridView.DataMember = $DataMember
 		
 		if ($Item -is [System.ComponentModel.IListSource]`
 		-or $Item -is [System.ComponentModel.IBindingList] -or $Item -is [System.ComponentModel.IBindingListView] )
 		{
 			$DataGridView.DataSource = $Item
 		}
 		else
 		{
 			$array = New-Object System.Collections.ArrayList
 			
 			if ($Item -is [System.Collections.IList])
 			{
 				$array.AddRange($Item)
 			}
 			else
 			{	
 				$array.Add($Item)	
 			}
 			$DataGridView.DataSource = $array
 		}
 		
 		$DataGridView.ResumeLayout()
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
 	
 	function Load-ListBox 
 	{
 	<#
 		.SYNOPSIS
 			This functions helps you load items into a ListBox or CheckedListBox.
 	
 		.DESCRIPTION
 			Use this function to dynamically load items into the ListBox control.
 	
 		.PARAMETER  ListBox
 			The ListBox control you want to add items to.
 	
 		.PARAMETER  Items
 			The object or objects you wish to load into the ListBox's Items collection.
 	
 		.PARAMETER  DisplayMember
 			Indicates the property to display for the items in this control.
 		
 		.PARAMETER  Append
 			Adds the item(s) to the ListBox without clearing the Items collection.
 		
 		.EXAMPLE
 			Load-ListBox $ListBox1 "Red", "White", "Blue"
 		
 		.EXAMPLE
 			Load-ListBox $listBox1 "Red" -Append
 			Load-ListBox $listBox1 "White" -Append
 			Load-ListBox $listBox1 "Blue" -Append
 		
 		.EXAMPLE
 			Load-ListBox $listBox1 (Get-Process) "ProcessName"
 	#>
 		Param (
 			[ValidateNotNull()]
 			[Parameter(Mandatory=$true)]
 			[System.Windows.Forms.ListBox]$ListBox,
 			[ValidateNotNull()]
 			[Parameter(Mandatory=$true)]
 			$Items,
 		    [Parameter(Mandatory=$false)]
 			[string]$DisplayMember,
 			[switch]$Append
 		)
 		
 		if(-not $Append)
 		{
 			$listBox.Items.Clear()	
 		}
 		
 		if($Items -is [System.Windows.Forms.ListBox+ObjectCollection])
 		{
 			$listBox.Items.AddRange($Items)
 		}
 		elseif ($Items -is [Array])
 		{
 			$listBox.BeginUpdate()
 			foreach($obj in $Items)
 			{
 				$listBox.Items.Add($obj)
 			}
 			$listBox.EndUpdate()
 		}
 		else
 		{
 			$listBox.Items.Add($Items)	
 		}
 	
 		$listBox.DisplayMember = $DisplayMember	
 	}
 	
 	$combobox1_SelectedIndexChanged={
 		
 	}
 	
 	$buttonCreate_Click={
 		
 	switch ($combobox1.SelectedItem) {
 		"Package Folder" {
 			$global:ObjectType = 2
 		}
 		"Application Folder" {
 			$global:ObjectType = 6000
 		}
 		value3 {
 			$global:ObjectType = 2
 		}
 		default {
 			$global:ObjectType = 2
 			}
 			
 		}
 	$Arguments = @{Name = $textbox1.Text; ObjectType = $ObjectType; ParentContainerNodeId = 0}
 	
 	$StoreResults = @{}
 	
 	Set-WmiInstance -Namespace "Root\SMS\Site_P01" -Class "SMS_ObjectContainerNode" -Arguments $Arguments -OutVariable StoreResults
 	if($Error)
 		{	
 			$labelProgress.Text = $Error[0].Exception.Message
 		}
 	Else {
 			$labelProgress.Text = "Folder Created Successfully. Results will be displayed soon...."
 		 }
 	
 	Load-DataGridView -DataGridView $datagridview1 -Item $StoreResults
 	
 	
 	
 		
 	
 	
 		
 	}
 	
 	
 	$datagridview1_CellContentClick=[System.Windows.Forms.DataGridViewCellEventHandler]{
 		
 	}
 	
 	$buttonBrowse_Click={
 		$openfiledialog1.ShowDialog()
 		$textbox2.Text = $OpenFileDialog1.FileName
 		$script:CSVFileName = $openfiledialog1.FileName
 	}
 	
 	$openfiledialog1_FileOk=[System.ComponentModel.CancelEventHandler]{
 		
 	}
 	
 	$datagridview2_CellContentClick=[System.Windows.Forms.DataGridViewCellEventHandler]{
 		
 	}
 	
 	$button1Create_Click={
 		$MultipleCreate = Import-Csv $CSVFileName
 		$TotalUserCount = $MultipleCreate.Count
 		for ($i=0;$i -lt $TotalUserCount;$i++) {		
 		    
 			$Arguments = @{Name = $MultipleCreate[$i].Name; ObjectType = $MultipleCreate[$i].ObjectType; ParentContainerNodeId = $MultipleCreate[$i].ParentContainerNodeId}
 			Set-WmiInstance -Namespace "Root\SMS\Site_P01" -Class "SMS_ObjectContainerNode" -Arguments $Arguments
 			$script:CustomObject2 = @{}
 			$CustomObject2[$i] = [pscustomobject]@{Name=$MultipleCreate[$i].Name; Created="True"}
 					
 			$progressbaroverlay1.Increment(($i + 1)/$TotalUserCount * 100)
 			$progressbaroverlay1.PerformStep()
 			
 		}
 		
 		Load-DataGridView -DataGridView $datagridview2 -Item $CustomObject2
 		
 			
 	}
 	
 	$progressbaroverlay1_Click={
 		
 	}
 	
 	#----------------------------------------------
 	#----------------------------------------------
 	
 	$Form_StateCorrection_Load=
 	{
 		$formConsoleFolderCreator.WindowState = $InitialFormWindowState
 	}
 	
 	$Form_Cleanup_FormClosed=
 	{
 		try
 		{
 			$progressbaroverlay1.remove_Click($progressbaroverlay1_Click)
 			$button1Create.remove_Click($button1Create_Click)
 			$datagridview2.remove_CellContentClick($datagridview2_CellContentClick)
 			$buttonBrowse.remove_Click($buttonBrowse_Click)
 			$datagridview1.remove_CellContentClick($datagridview1_CellContentClick)
 			$buttonCreate.remove_Click($buttonCreate_Click)
 			$combobox1.remove_SelectedIndexChanged($combobox1_SelectedIndexChanged)
 			$formConsoleFolderCreator.remove_Load($formConsoleFolderCreator_Load)
 			$openfiledialog1.remove_FileOk($openfiledialog1_FileOk)
 			$formConsoleFolderCreator.remove_Load($Form_StateCorrection_Load)
 			$formConsoleFolderCreator.remove_FormClosed($Form_Cleanup_FormClosed)
 		}
 		catch [Exception]
 		{ }
 	}
 
 	#----------------------------------------------
 	#----------------------------------------------
 	#
 	#
 	$formConsoleFolderCreator.Controls.Add($progressbaroverlay1)
 	$formConsoleFolderCreator.Controls.Add($button1Create)
 	$formConsoleFolderCreator.Controls.Add($datagridview2)
 	$formConsoleFolderCreator.Controls.Add($buttonBrowse)
 	$formConsoleFolderCreator.Controls.Add($textbox2)
 	$formConsoleFolderCreator.Controls.Add($labelCSVFile)
 	$formConsoleFolderCreator.Controls.Add($labelProgress)
 	$formConsoleFolderCreator.Controls.Add($datagridview1)
 	$formConsoleFolderCreator.Controls.Add($buttonCreate)
 	$formConsoleFolderCreator.Controls.Add($combobox1)
 	$formConsoleFolderCreator.Controls.Add($textbox1)
 	$formConsoleFolderCreator.Controls.Add($labelName)
 	$formConsoleFolderCreator.ClientSize = '522, 603'
 	$formConsoleFolderCreator.Name = "formConsoleFolderCreator"
 	$formConsoleFolderCreator.Text = "Console Folder Creator"
 	$formConsoleFolderCreator.add_Load($formConsoleFolderCreator_Load)
 	#
 	#
 	$progressbaroverlay1.Location = '29, 387'
 	$progressbaroverlay1.Name = "progressbaroverlay1"
 	$progressbaroverlay1.Size = '464, 27'
 	$progressbaroverlay1.TabIndex = 11
 	$progressbaroverlay1.add_Click($progressbaroverlay1_Click)
 	#
 	#
 	$button1Create.Location = '169, 558'
 	$button1Create.Name = "button1Create"
 	$button1Create.Size = '152, 27'
 	$button1Create.TabIndex = 10
 	$button1Create.Text = "Create"
 	$button1Create.UseVisualStyleBackColor = $True
 	$button1Create.add_Click($button1Create_Click)
 	#
 	#
 	$datagridview2.ColumnHeadersHeightSizeMode = 'AutoSize'
 	$datagridview2.Location = '30, 425'
 	$datagridview2.Name = "datagridview2"
 	$datagridview2.Size = '464, 121'
 	$datagridview2.TabIndex = 9
 	$datagridview2.add_CellContentClick($datagridview2_CellContentClick)
 	#
 	#
 	$buttonBrowse.Location = '421, 347'
 	$buttonBrowse.Name = "buttonBrowse"
 	$buttonBrowse.Size = '73, 21'
 	$buttonBrowse.TabIndex = 8
 	$buttonBrowse.Text = "Browse"
 	$buttonBrowse.UseVisualStyleBackColor = $True
 	$buttonBrowse.add_Click($buttonBrowse_Click)
 	#
 	#
 	$textbox2.Location = '83, 348'
 	$textbox2.Name = "textbox2"
 	$textbox2.Size = '319, 20'
 	$textbox2.TabIndex = 7
 	#
 	#
 	$labelCSVFile.Location = '27, 352'
 	$labelCSVFile.Name = "labelCSVFile"
 	$labelCSVFile.Size = '71, 27'
 	$labelCSVFile.TabIndex = 6
 	$labelCSVFile.Text = "CSV File :"
 	#
 	#
 	$labelProgress.Location = '26, 107'
 	$labelProgress.Name = "labelProgress"
 	$labelProgress.Size = '468, 29'
 	$labelProgress.TabIndex = 5
 	#
 	#
 	$datagridview1.AutoSizeColumnsMode = 'ColumnHeader'
 	$datagridview1.ColumnHeadersHeightSizeMode = 'AutoSize'
 	$datagridview1.Location = '24, 154'
 	$datagridview1.Name = "datagridview1"
 	$datagridview1.Size = '471, 150'
 	$datagridview1.TabIndex = 4
 	$datagridview1.add_CellContentClick($datagridview1_CellContentClick)
 	#
 	#
 	$buttonCreate.Location = '408, 57'
 	$buttonCreate.Name = "buttonCreate"
 	$buttonCreate.Size = '88, 21'
 	$buttonCreate.TabIndex = 3
 	$buttonCreate.Text = "&Create"
 	$buttonCreate.UseVisualStyleBackColor = $True
 	$buttonCreate.add_Click($buttonCreate_Click)
 	#
 	#
 	$combobox1.FormattingEnabled = $True
 	[void]$combobox1.Items.Add("Package Folder")
 	[void]$combobox1.Items.Add("Query Folder")
 	[void]$combobox1.Items.Add("Software Metering Folder")
 	[void]$combobox1.Items.Add("Operating System Installer Folder")
 	[void]$combobox1.Items.Add("Application Folder")
 	$combobox1.Location = '220, 57'
 	$combobox1.Name = "combobox1"
 	$combobox1.Size = '182, 21'
 	$combobox1.TabIndex = 2
 	$combobox1.add_SelectedIndexChanged($combobox1_SelectedIndexChanged)
 	#
 	#
 	$textbox1.Location = '60, 58'
 	$textbox1.Name = "textbox1"
 	$textbox1.Size = '154, 20'
 	$textbox1.TabIndex = 1
 	$textbox1.Text = "Enter the Name of the Folder"
 	#
 	#
 	$labelName.Location = '18, 62'
 	$labelName.Name = "labelName"
 	$labelName.Size = '47, 19'
 	$labelName.TabIndex = 0
 	$labelName.Text = "Name"
 	#
 	#
 	$openfiledialog1.FileName = "openfiledialog1"
 	$openfiledialog1.Filter = "CSV Files | *.csv"
 	$openfiledialog1.InitialDirectory = "C:\Scripts\Forms"
 	$openfiledialog1.add_FileOk($openfiledialog1_FileOk)
 
 	#----------------------------------------------
 
 	$InitialFormWindowState = $formConsoleFolderCreator.WindowState
 	$formConsoleFolderCreator.add_Load($Form_StateCorrection_Load)
 	$formConsoleFolderCreator.add_FormClosed($Form_Cleanup_FormClosed)
 	return $formConsoleFolderCreator.ShowDialog()
 
 
 if((OnApplicationLoad) -eq $true)
 {
 	Call-CreateFolder_pff | Out-Null
 	OnApplicationExit
 }
`

