---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 2267
Published Date: 
Archived Date: 2010-09-26t20
---

#  - 

## Description

the easy migration script enables you to manually vmotion and svmotion virtual machines very quickly and easily. you don’t have to specify the resource pools and it reduces the amount of mouse clicks required. there is also an undo feature.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function GenerateForm {
 ########################################################################
 #
 ########################################################################
 
 [reflection.assembly]::loadwithpartialname("System.Drawing") | Out-Null
 [reflection.assembly]::loadwithpartialname("System.Windows.Forms") | Out-Null
 
 $MigrationForm = New-Object System.Windows.Forms.Form
 $Header = new-object System.Windows.Forms.Label
 $MigrateButton = New-Object System.Windows.Forms.Button
 $ClusterListBox = New-Object System.Windows.Forms.ListBox
 $SourceHostListBox = New-Object System.Windows.Forms.ListBox
 $SourceDBListBox = New-Object System.Windows.Forms.CheckedListBox
 $SourceVMListBox = New-Object System.Windows.Forms.CheckedListBox
 $DestinationListBox = New-Object System.Windows.Forms.ListBox
 $vCenterLabel = New-Object System.Windows.Forms.Label
 $VCTextBox = New-Object System.Windows.Forms.TextBox
 $LoginButton = New-Object System.Windows.Forms.Button
 $ClearMsgButton = New-Object System.Windows.Forms.Button
 $UndoButton = New-Object System.Windows.Forms.Button
 $UndoALLButton = New-Object System.Windows.Forms.Button
 $SaveTaskButton = New-Object System.Windows.Forms.Button
 $ClusterLabel = New-Object System.Windows.Forms.Label
 $HostToHostButton = New-Object System.Windows.Forms.Button
 $VMToHostButton = New-Object System.Windows.Forms.Button
 $DBtoDBButton = New-Object System.Windows.Forms.Button
 $VMtoDBButton = New-Object System.Windows.Forms.Button
 $SourceButton = New-Object System.Windows.Forms.Button
 $DestinationButton = New-Object System.Windows.Forms.Button
 $ResetButton = New-Object System.Windows.Forms.Button
 $ChangeVCButton = New-Object System.Windows.Forms.Button
 $CloseButton = New-Object System.Windows.Forms.Button
 $MessagesLabel = New-Object System.Windows.Forms.Label
 $MessagesListBox = New-Object System.Windows.Forms.ListBox
 $TaskLabel = New-Object System.Windows.Forms.Label
 $TaskListBox = New-Object System.Windows.Forms.ListBox
 $InitialFormWindowState = New-Object System.Windows.Forms.FormWindowState
 
 #----------------------------------------------
 #----------------------------------------------
 
 $LoginButton_Click= 
 {
 If ($VCTextBox.Text -eq "")
 {
 [reflection.assembly]::loadwithpartialname('system.windows.forms') 
 [system.Windows.Forms.MessageBox]::show('Please Enter a Valid vCenter Server.')
 }
 Else
 {
 if ((Get-PSSnapin "VMware.VimAutomation.Core" -ErrorAction SilentlyContinue) -eq $null) {
 Add-PSSnapin "VMware.VimAutomation.Core"
 } 
 Connect-VIServer $VCTextBox.Text
 
 $ClusterListBox.Enabled = $true
 
 Get-Cluster | % { [void]$ClusterListBox.items.add($_.Name) }
 
 If ($ClusterListBox.items -ne $null)
 {
 $LoginButton.Enabled = $false
 $VCTextBox.Enabled = $false
 $HostToHostButton.Enabled = $true
 $VMToHostButton.Enabled = $true
 $DBtoDBButton.Enabled = $true
 $VMtoDBButton.Enabled = $true
 $ResetButton.Enabled = $true
 $ChangeVCButton.Enabled = $true
 
 #$logfile = "C:\TEMP\EasyMigrate{0:dd-MM-yyyy_HHmmss}" -f (Get-Date) + ".log"
 #$logheader = "Easy Migration Tool v1.0 - Log generated on " + (Get-Date)
 }
 
 $OnLoadForm_StateCorrection=
 	$MigrationForm.WindowState = $InitialFormWindowState
 }
 }
 }
 
 $HostToHostButton_Click= 
 {
 
 $ClusterName = $ClusterListBox.SelectedItem
 
 If ($ClusterName -eq $null)
 {
 [reflection.assembly]::loadwithpartialname('system.windows.forms') 
 [system.Windows.Forms.MessageBox]::show('Please Select a Cluster.')
 }
 Else
 {
 $SourceHostListBox.Enabled = $true
 $SourceHostListBox.Items.Clear()
 
 Get-Cluster -Name $ClusterName | Get-VMHost | % { [void]$SourceHostListBox.items.add($_.Name) }
 
 $ClusterListBox.Enabled = $false
 $HostToHostButton.Enabled = $false
 $VMToHostButton.Enabled = $false
 $DBtoDBButton.Enabled = $false
 $VMtoDBButton.Enabled = $false
 $SourceButton.Text = 'Source - Select One Host'
 $DestinationButton.Text = 'Destination Host'
 $SourceButton.Enabled = $true
 $HostToHostButton.BackColor = [System.Drawing.Color]::lightblue
 
 $OnLoadForm_StateCorrection=
 	$MigrationForm.WindowState = $InitialFormWindowState
 }
 }
 }
 
 $VMToHostButton_Click= 
 {
 
 $ClusterName = $ClusterListBox.SelectedItem
 
 If ($ClusterName -eq $null)
 {
 [reflection.assembly]::loadwithpartialname('system.windows.forms') 
 [system.Windows.Forms.MessageBox]::show('Please Select a Cluster.')
 }
 Else
 {
 $MigrationForm.Controls.Remove($SourceHostListBox)
 $MigrationForm.Controls.Add($SourceVMListBox)
 $SourceVMListBox.Enabled = $true
 $SourceVMListBox.Items.Clear()
 
 Get-Cluster -Name $ClusterName | Get-VM | % { [void]$SourceVMListBox.items.add($_.Name) }
 
 $ClusterListBox.Enabled = $false
 $HostToHostButton.Enabled = $false
 $VMToHostButton.Enabled = $false
 $DBtoDBButton.Enabled = $false
 $VMtoDBButton.Enabled = $false
 $SourceButton.Text = 'Source - Select VMs'
 $DestinationButton.Text = 'Destination Host'
 $SourceButton.Enabled = $true
 $VMToHostButton.BackColor = [System.Drawing.Color]::lightblue
 
 $OnLoadForm_StateCorrection=
 	$MigrationForm.WindowState = $InitialFormWindowState
 }
 }
 }
 
 $DBtoDBButton_Click= 
 {
 
 $ClusterName = $ClusterListBox.SelectedItem
 
 If ($ClusterName -eq $null)
 {
 [reflection.assembly]::loadwithpartialname('system.windows.forms') 
 [system.Windows.Forms.MessageBox]::show('Please Select a Cluster.')
 }
 Else
 {
 $MigrationForm.Controls.Remove($SourceHostListBox)
 $MigrationForm.Controls.Add($SourceDBListBox)
 $SourceDBListBox.Enabled = $true
 $SourceDBListBox.Items.Clear()
 
 Get-Datastore -Name *$ClusterName* | Where-Object {$_.Name -notlike "*SCR01"} | Where-Object {$_.Name -notlike "DAS*"} | % { [void]$SourceDBListBox.items.add($_.Name) }
 
 $ClusterListBox.Enabled = $false
 $HostToHostButton.Enabled = $false
 $VMToHostButton.Enabled = $false
 $DBtoDBButton.Enabled = $false
 $VMtoDBButton.Enabled = $false
 $SourceButton.Text = 'Source - Select a Datastore'
 $DestinationButton.Text = 'Destination Datastore'
 $SourceButton.Enabled = $true
 $DBtoDBButton.BackColor = [System.Drawing.Color]::lightblue
 
 $OnLoadForm_StateCorrection=
 	$MigrationForm.WindowState = $InitialFormWindowState
 }
 }
 }
 
 $VMToDBButton_Click= 
 {
 
 $ClusterName = $ClusterListBox.SelectedItem
 
 If ($ClusterName -eq $null)
 {
 [reflection.assembly]::loadwithpartialname('system.windows.forms') 
 [system.Windows.Forms.MessageBox]::show('Please Select a Cluster.')
 }
 Else
 {
 $MigrationForm.Controls.Remove($SourceHostListBox)
 $MigrationForm.Controls.Add($SourceVMListBox)
 $SourceVMListBox.Enabled = $true
 $SourceVMListBox.Items.Clear()
 
 Get-Cluster -Name $ClusterName | Get-VM | % { [void]$SourceVMListBox.items.add($_.Name) }
 
 $ClusterListBox.Enabled = $false
 $HostToHostButton.Enabled = $false
 $VMToHostButton.Enabled = $false
 $DBtoDBButton.Enabled = $false
 $VMtoDBButton.Enabled = $false
 $SourceButton.Text = 'Source - Select VMs'
 $DestinationButton.Text = 'Destination Datastore'
 $SourceButton.Enabled = $true
 $VMtoDBButton.BackColor = [System.Drawing.Color]::lightblue
 
 $OnLoadForm_StateCorrection=
 	$MigrationForm.WindowState = $InitialFormWindowState
 }
 }
 }
 
 $SourceButton_OnClick= 
 {
 
 $ClusterName = $ClusterListBox.SelectedItem
 
 If ($SourceButton.Text -eq "Source - Select One Host")
 {
 	$SourceHostname = $SourceHostListBox.SelectedItem
 
 	If ($SourceHostname -eq $null)
 	{	
 	[reflection.assembly]::loadwithpartialname('system.windows.forms') 
 	[system.Windows.Forms.MessageBox]::show('Please Select a Source Host.')
 	}
 	Else
 	{
 	Get-Cluster -Name $ClusterName | Get-VMHost | Where-Object {$_.Name -notlike "$SourceHostname"} | % { [void]$DestinationListBox.items.add($_.Name) }
 
 	$SourceHostListBox.Enabled = $false
 	$SourceButton.Enabled = $false
 	$DestinationListBox.Enabled = $true
 	$DestinationButton.Enabled = $true
 
 	$OnLoadForm_StateCorrection=
 			$MigrationForm.WindowState = $InitialFormWindowState
 		}
 	}
 }
 
 If ($SourceButton.Text -eq "Source - Select VMs")
 {
 	$SourceVMname = $SourceVMListBox.SelectedItem
 	
 	If ($SourceVMname -eq $null)
 	{	
 	[reflection.assembly]::loadwithpartialname('system.windows.forms') 
 	[system.Windows.Forms.MessageBox]::show('Please Select One or More VMs.')
 	}
 	Else
 	{
 	if ($DestinationButton.Text -like "Destination Host")
 	{
 	Get-Cluster -Name $ClusterName | Get-VMHost | % { [void]$DestinationListBox.items.add($_.Name) }
 	}
 	Else
 	{
 	Get-Datastore -Name *$ClusterName* | Where-Object {$_.Name -notlike "*SCR01"} | Where-Object {$_.Name -notlike "DAS*"} | % { [void]$DestinationListBox.items.add($_.Name) }
 	}
 	
 	$DestinationListBox.Enabled = $true
 	$SourceVMListBox.Enabled = $false
 	$SourceButton.Enabled = $false
 	$DestinationButton.Enabled = $true
 	
 	$OnLoadForm_StateCorrection=
 			$MigrationForm.WindowState = $InitialFormWindowState
 		}
 	}
 }
 
 If ($SourceButton.Text -like "Source - Select a Datastore")
 {
 	$SourceDBname = $SourceDBListBox.SelectedItem
 
 	If ($SourceDBname -eq $null)
 	{	
 	[reflection.assembly]::loadwithpartialname('system.windows.forms') 
 	[system.Windows.Forms.MessageBox]::show('Please Select One Datastore.')
 	}
 	Else
 	{
 	Get-Datastore -Name *$ClusterName* | Where-Object {$_.Name -notlike "*SCR01"} | Where-Object {$_.Name -notlike "DAS*"} | Where-Object {$_.Name -notlike "$SourceDBname"} | % { [void]$DestinationListBox.items.add($_.Name) }
 
 	$SourceDBListBox.Enabled = $false
 	$SourceButton.Enabled = $false
 	$DestinationListBox.Enabled = $true
 	$DestinationButton.Enabled = $true
 
 	$OnLoadForm_StateCorrection=
 			$MigrationForm.WindowState = $InitialFormWindowState
 		}
 	}
 }
 
 }
 
 $DestinationButton_OnClick= 
 {
 
 $DestinationName = $DestinationListBox.SelectedItem
 
 If ($DestinationName -eq $null)
 {
 [reflection.assembly]::loadwithpartialname('system.windows.forms') 
 [system.Windows.Forms.MessageBox]::show('Please Select a Destination.')
 }
 Else
 {
 $DestinationListBox.Enabled = $false
 $DestinationButton.Enabled = $false
 $MigrateButton.Enabled = $true
 
 $OnLoadForm_StateCorrection=
 	$MigrationForm.WindowState = $InitialFormWindowState
 }
 }
 }
 
 $MigrateButton_OnClick= 
 {
 
 If ($SourceButton.Text -eq "Source - Select One Host")
 {
 $SourceHostname = $SourceHostListBox.SelectedItem
 $DestinationName = $DestinationListBox.SelectedItem
 $VMs = Get-VMHost $SourceHostname | Get-VM
 
 	foreach($VM in $VMs)
 	{
 	Get-VM -Name $VM | Move-VM -Destination (Get-VMHost $DestinationName)
 	
 	$d = Get-Date
 	$TaskLine = "$d - $VM migrated successfully from $SourceHostName to $DestinationName"
 	[void]$TaskListBox.items.add($TaskLine)
 	}
 	
 	$SourceVMListBox.enabled = $true
 	$SourceButton.enabled = $true
 	$MigrateButton.Enabled = $false
 	$DestinationListBox.Items.Clear()
 }
 
 If ($SourceButton.Text -eq "Source - Select VMs")
 {
 	$DestinationName = $DestinationListBox.SelectedItem
 	
 	If($DestinationButton.Text -eq "Destination Host")
 	{
 	foreach($VM in $SourceVMListBox.CheckedItems)
 	{
 	$SourceName = Get-VM -Name $VM | Get-VMHost | % {$_.name}
 	If($SourceName -ne $DestinationName)
 	{
 	Get-VM -Name $VM | Move-VM -Destination (Get-VMHost $DestinationName)
 	
 	$d = Get-Date
 	$line = "$d - $VM migrated successfully from $SourceName to $DestinationName"
 	[void]$TaskListBox.items.add($line)
 	}
 	Else
 	{
 	$d = Get-Date
 	$MsgLine = "$d - Cannot migrate $VM because the source host and the destination host are the same."
 	[void]$MessagesListBox.items.add($MsgLine)
 	}
 	}
 	}
 	Else
 	{
 	foreach($VM in $SourceVMListBox.CheckedItems)
 	{
 	$VMsize = ((Get-VM -Name $VM | Get-HardDisk | Measure-Object -property CapacityKB -Sum).Sum)/1024
 	$DBFreeSpace = Get-Datastore -Name $DestinationName | %{$_.FreeSpaceMB}
 	$DBReserved = Get-Datastore -Name $DestinationName | %{$_.CapacityMB * 0.2}
 	$DBAvailableSpace = $DBFreeSpace - $DBReserved
 		
 	If ($VMsize -gt $DBAvailableSpace)
 	{
 	$VMSizeGB = "{0:N1}GB" -f ($VMsize / 1024)
 	$DBFreeSpaceGB = "{0:N1} GB" -f ($DBFreeSpace / 1024)
 	$DBAvailableSpaceGB = "{0:N1} GB" -f ($DBAvailableSpace / 1024)
 	
 	$d = Get-Date
 	$MsgLine = "$d - Cannot migrate $VM (Size = $VMSizeGB) because there is insufficient disk space on $DestinationName (Available = $DBAvailableSpaceGB)."
 	[void]$MessagesListBox.items.add($MsgLine)
 	}
 	Else
 	{
 	$SourceName = Get-VM -Name $VM | Get-Datastore | % {$_.name}
 	If($SourceName -ne $DestinationName)
 	{
 		Get-VM -Name $VM | Move-VM -Datastore ($DestinationName)
 		
 		$d = Get-Date
 		$TaskLine = "$d - $VM migrated successfully from $SourceName to $DestinationName"
 		[void]$TaskListBox.items.add($TaskLine)
 	}
 	Else
 	{
 		$d = Get-Date
 		$MsgLine = "$d - Cannot migrate $VM because the source datastore and the destination datastore are the same."
 		[void]$MessagesListBox.items.add($MsgLine)
 	}
 	}
 	}
 	}
 	
 	$SourceVMListBox.enabled = $true
 	$SourceButton.enabled = $true
 	$MigrateButton.Enabled = $false
 	$DestinationListBox.Items.Clear()
 }
 
 If ($SourceButton.Text -eq "Source - Select a Datastore")
 {
 $SourceDBname = $SourceDBListBox.SelectedItem
 $DestinationName = $DestinationListBox.SelectedItem
 
 $VMs = Get-Datastore -Name $SourceDBname | Get-VM
 
 	If($VMs.count -gt 0)
 	{
 	foreach($VM in $VMs)
 	{
 	$VMsize = ((Get-VM -Name $VM | Get-HardDisk | Measure-Object -property CapacityKB -Sum).Sum)/1024
 	$DBFreeSpace = Get-Datastore -Name $DestinationName | %{$_.FreeSpaceMB}
 	$DBReserved = Get-Datastore -Name $DestinationName | %{$_.CapacityMB * 0.2}
 	$DBAvailableSpace = $DBFreeSpace - $DBReserved
 		
 	If ($VMsize -gt $DBFreeSpace)
 	{
 	$VMSizeGB = "{0:N1}GB" -f ($VMsize / 1024)
 	$DBFreeSpaceGB = "{0:N1} GB" -f ($DBFreeSpace / 1024)
 	$DBAvailableSpaceGB = "{0:N1} GB" -f ($DBAvailableSpace / 1024)
 	
 	$d = Get-Date
 	$MsgLine = "$d - Cannot migrate $VM (Size = $VMSizeGB) because there is insufficient disk space on $DestinationName (Available = $DBAvailableSpaceGB)."
 	[void]$MessagesListBox.items.add($MsgLine)
 	}
 	Else
 	{
 	Get-VM -Name $VM | Move-VM -Datastore ($DestinationName)
 	
 	$d = Get-Date
 	$line = "$d - $VM migrated successfully from $SourceDBName to $DestinationName"
 	[void]$TaskListBox.items.add($line)
 	}
 	}
 	}
 	Else
 	{
 	$d = Get-Date
 	$MsgLine = "$d - No VM is present on the source datastore $SourceDBname."
 	[void]$MessagesListBox.items.add($MsgLine)
 	}
 
 	$SourceDBListBox.enabled = $true
 	$SourceButton.enabled = $true
 	$DestinationListBox.Items.Clear()
 	$MigrateButton.Enabled = $false
 }
 }
 
 $RESETButton_OnClick= 
 {
 
 	$ClusterListBox.Enabled = $true
 	$HostToHostButton.Enabled = $true
 	$VMToHostButton.Enabled = $true
 	$DBtoDBButton.Enabled = $true
 	$VMtoDBButton.Enabled = $true
 	$SourceHostListBox.Items.Clear()
 	$SourceDBListBox.Items.Clear()
 	$SourceVMListBox.Items.Clear()
 	$SourceButton.Enabled = $false
 	$SourceButton.Text = 'Source'
 	$DestinationListBox.Items.Clear()
 	$DestinationButton.Enabled = $false
 	$DestinationButton.Text = 'Destination'
 	$MigrateButton.Enabled = $false
 	$MigrationForm.Controls.Remove($SourceHostListBox)
 	$MigrationForm.Controls.Remove($SourceVMListBox)
 	$MigrationForm.Controls.Remove($SourceDBListBox)
 	$MigrationForm.Controls.Add($SourceHostListBox)
 	$HostToHostButton.BackColor = [System.Drawing.Color]::Transparent
 	$VMToHostButton.BackColor = [System.Drawing.Color]::Transparent
 	$DBtoDBButton.BackColor = [System.Drawing.Color]::Transparent
 	$VMtoDBButton.BackColor = [System.Drawing.Color]::Transparent
 }
 
 $ChangeVCButton_OnClick= 
 {
 
 	$VCTextBox.Text = ""
 	$VCTextBox.Enabled = $true
 	$LoginButton.Enabled = $true
 	$ClusterListBox.Items.Clear()
 	$ClusterListBox.Enabled = $false
 	$HostToHostButton.Enabled = $false
 	$VMToHostButton.Enabled = $false
 	$DBtoDBButton.Enabled = $false
 	$VMtoDBButton.Enabled = $false
 	$SourceHostListBox.Items.Clear()
 	$SourceDBListBox.Items.Clear()
 	$SourceVMListBox.Items.Clear()
 	$SourceButton.Enabled = $false
 	$SourceButton.Text = 'Source'
 	$DestinationListBox.Items.Clear()
 	$DestinationButton.Enabled = $false
 	$DestinationButton.Text = 'Destination'
 	$MigrateButton.Enabled = $false
 	$ResetButton.Enabled = $false
 	$ChangeVCButton.Enabled = $false
 	$MessagesListBox.Items.Clear()
 	$TaskListBox.Items.Clear()
 	$MigrationForm.Controls.Remove($SourceHostListBox)
 	$MigrationForm.Controls.Remove($SourceVMListBox)
 	$MigrationForm.Controls.Remove($SourceDBListBox)
 	$MigrationForm.Controls.Add($SourceHostListBox)
 	$HostToHostButton.BackColor = [System.Drawing.Color]::Transparent
 	$VMToHostButton.BackColor = [System.Drawing.Color]::Transparent
 	$DBtoDBButton.BackColor = [System.Drawing.Color]::Transparent
 	$VMtoDBButton.BackColor = [System.Drawing.Color]::Transparent
 	
 	Disconnect-VIServer -Confirm:$false
 }
 
 $CloseButton_OnClick= 
 {
 If ($VCTextBox.Text -ne "")
 {
 	Disconnect-VIServer -Confirm:$false
 }
 	$MigrationForm.close()
 }
 
 $ClearMsgButton_Click= 
 {
 If($MessagesListBox.items.count -eq 0)
 {
 [reflection.assembly]::loadwithpartialname('system.windows.forms') 
 [system.Windows.Forms.MessageBox]::show('No messages to clear.')
 }
 Else
 {
 $MessagesListBox.items.clear()
 }
 }
 
 $UndoButton_Click= 
 {
 
 If ($TaskListBox.Items.Count -eq 0)
 {
 [reflection.assembly]::loadwithpartialname('system.windows.forms') 
 [system.Windows.Forms.MessageBox]::show('The task list is empty.')
 }
 Else
 {
 If ($TaskListBox.SelectedItems.Count -eq 0)
 {
 [reflection.assembly]::loadwithpartialname('system.windows.forms') 
 [system.Windows.Forms.MessageBox]::show('Please select one or more tasks.')
 }
 Else
 {
 	foreach($Task in $TaskListBox.SelectedItems)
 	{
 		$Task = $Task.split(" ")
 		$VM = $Task[3]
 		$SourceName = $Task[9]
 		$DestinationName = $Task[7]
 
 		Get-VM -Name $VM | Move-VM -Destination (Get-VMhost $DestinationName)
 	}
 	
 	While($TaskListBox.SelectedItems.Count -gt 0)
 	{
     	$TaskListBox.Items.Remove($TaskListBox.SelectedItems[0])
 	}
 }
 }
 }
 
 $UndoALLButton_Click= 
 {
 
 If ($TaskListBox.Items.Count -eq 0)
 {
 [reflection.assembly]::loadwithpartialname('system.windows.forms') 
 [system.Windows.Forms.MessageBox]::show('The task list is empty.')
 }
 Else
 {
 	foreach($Task in $TaskListBox.Items)
 	{
 		$Task = $Task.split(" ")
 		$VM = $Task[3]
 		$SourceName = $Task[9]
 		$DestinationName = $Task[7]
 
 		Get-VM -Name $VM | Move-VM -Destination (Get-VMhost $DestinationName)
 
 	}
 	
 	$TaskListBox.Items.Clear()
 }
 }
 
 $SaveTaskButton_Click= 
 {
 
 If ($TaskListBox.items.count -eq 0)
 {
 [reflection.assembly]::loadwithpartialname('system.windows.forms') 
 [system.Windows.Forms.MessageBox]::show('Nothing to save to log file.')
 }
 Else
 {
 $SaveFileD = new-object System.Windows.Forms.SaveFileDialog
 $SaveFileD.Filter = "All files (*.*)|*.*|log files (*.log)|*.log";
 $SaveFileD.Title = "Save to a log file";
 $SaveFileD.FilterIndex = 2;
 $SaveFileD.RestoreDirectory = $true;
 if ($SaveFileD.ShowDialog() -eq [System.Windows.Forms.DialogResult]::OK)
 {
 $FileName = $SaveFileD.FileName
 $Tasks = $TaskListBox.items
 $logheader = "Easy Migration Tool v1.0 - Log generated on " + (Get-Date)
 
 Out-File -InputObject $logheader -FilePath $Filename
 Out-File -InputObject "" -Append -FilePath $FileName
 Out-File -InputObject $Tasks -Append -FilePath $FileName
 }
 }
 }
 
 #----------------------------------------------
 $MigrationForm.Text = 'Easy Migration Tool v1.0'
 $MigrationForm.Font = new-object System.Drawing.Font("Tahoma",8,[System.Drawing.FontStyle]::Bold)
 $MigrationForm.Name = 'EasyMigrationForm'
 $MigrationForm.StartPosition = "CenterScreen"
 $MigrationForm.FormBorderStyle = "FixedDialog"
 $MigrationForm.MaximizeBox = $false
 $MigrationForm.MinimizeBox = $false
 $MigrationForm.DataBindings.DefaultDataSourceUpdateMode = 0
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 1030
 $System_Drawing_Size.Height = 500
 $MigrationForm.ClientSize = $System_Drawing_Size
 
 $Header.Font = new-object System.Drawing.Font("Tahoma",10,[System.Drawing.FontStyle]::Bold)
 $Header.TextAlign = [System.Drawing.ContentAlignment]::MiddleLeft
 $Header.Text = 'Easy Migration Tool v1.0'
 $Header.Location = new-object System.Drawing.Point(150,15)
 $Header.Size = new-object System.Drawing.Size(230, 20)
 
 $MigrationForm.Controls.Add($Header)
 
 $vCenterLabel.TabIndex = 2
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 95
 $System_Drawing_Size.Height = 20
 $vCenterLabel.Size = $System_Drawing_Size
 $vCenterLabel.Font = new-object System.Drawing.Font("Tahoma",8,[System.Drawing.FontStyle]::Bold)
 $vCenterLabel.Text = 'vCenter Server:'
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 15
 $System_Drawing_Point.Y = 50
 $vCenterLabel.Location = $System_Drawing_Point
 $vCenterLabel.DataBindings.DefaultDataSourceUpdateMode = 0
 $vCenterLabel.Name = 'vCenterLabel'
 
 $MigrationForm.Controls.Add($vCenterLabel)
 
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 120
 $System_Drawing_Size.Height = 20
 $VCTextBox.Size = $System_Drawing_Size
 $VCTextBox.Font = new-object System.Drawing.Font("Tahoma",8)
 $VCTextBox.DataBindings.DefaultDataSourceUpdateMode = 0
 $VCTextBox.Name = 'VCTextBox'
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 110
 $System_Drawing_Point.Y = 50
 $VCTextBox.Location = $System_Drawing_Point
 $VCTextBox.TabIndex = 1
 
 $MigrationForm.Controls.Add($VCTextBox)
 
 $LoginButton.TabIndex = 0
 $LoginButton.Name = 'LoginButton'
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 95
 $System_Drawing_Size.Height = 20
 $LoginButton.Size = $System_Drawing_Size
 $LoginButton.Font = new-object System.Drawing.Font("Tahoma",8)
 $LoginButton.UseVisualStyleBackColor = $True
 
 $LoginButton.Text = 'Login'
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 250
 $System_Drawing_Point.Y = 50
 $LoginButton.Location = $System_Drawing_Point
 $LoginButton.DataBindings.DefaultDataSourceUpdateMode = 0
 $LoginButton.add_Click($LoginButton_Click)
 
 $MigrationForm.Controls.Add($LoginButton)
 
 $ChangeVCButton.TabIndex = 19
 $ChangeVCButton.Name = 'ChangeVCButton'
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 95
 $System_Drawing_Size.Height = 20
 $ChangeVCButton.Size = $System_Drawing_Size
 $ChangeVCButton.Font = new-object System.Drawing.Font("Tahoma",8)
 $ChangeVCButton.UseVisualStyleBackColor = $True
 
 $ChangeVCButton.Text = 'Change vCenter'
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 360
 $System_Drawing_Point.Y = 50
 $ChangeVCButton.Location = $System_Drawing_Point
 $ChangeVCButton.DataBindings.DefaultDataSourceUpdateMode = 0
 $ChangeVCButton.add_Click($ChangeVCButton_OnClick)
 $ChangeVCButton.Enabled = $false
 
 $MigrationForm.Controls.Add($ChangeVCButton)
 
 $ClusterLabel.TabIndex = 5
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 85
 $System_Drawing_Size.Height = 20
 $ClusterLabel.Size = $System_Drawing_Size
 $ClusterLabel.Font = new-object System.Drawing.Font("Tahoma",8,[System.Drawing.FontStyle]::Bold)
 $ClusterLabel.Text = 'Select Cluster:'
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 20
 $System_Drawing_Point.Y = 90
 $ClusterLabel.Location = $System_Drawing_Point
 $ClusterLabel.DataBindings.DefaultDataSourceUpdateMode = 0
 $ClusterLabel.Name = 'ClusterLabel'
 
 $MigrationForm.Controls.Add($ClusterLabel)
 
 $ClusterListBox.FormattingEnabled = $True
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 120
 $System_Drawing_Size.Height = 100
 $ClusterListBox.Size = $System_Drawing_Size
 $ClusterListBox.Font = new-object System.Drawing.Font("Tahoma",8)
 $ClusterListBox.DataBindings.DefaultDataSourceUpdateMode = 0
 $ClusterListBox.Name = 'ClusterListBox'
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 110
 $System_Drawing_Point.Y = 90
 $ClusterListBox.Location = $System_Drawing_Point
 $ClusterListBox.MultiColumn = $false
 $ClusterListBox.TabIndex = 5
 $ClusterListBox.Sorted = $True
 
 $MigrationForm.Controls.Add($ClusterListBox)
 
 $HostToHostButton.TabIndex = 3
 $HostToHostButton.Name = 'HostToHostButton'
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 130
 $System_Drawing_Size.Height = 20
 $HostToHostButton.Size = $System_Drawing_Size
 $HostToHostButton.Font = new-object System.Drawing.Font("Tahoma",8)
 $HostToHostButton.UseVisualStyleBackColor = $True
 
 $HostToHostButton.Text = 'Host to Host'
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 250
 $System_Drawing_Point.Y = 90
 $HostToHostButton.Location = $System_Drawing_Point
 $HostToHostButton.DataBindings.DefaultDataSourceUpdateMode = 0
 $HostToHostButton.add_Click($HostToHostButton_Click)
 $HostToHostButton.Enabled = $false
 
 $MigrationForm.Controls.Add($HostToHostButton)
 
 $VMToHostButton.TabIndex = 3
 $VMToHostButton.Name = 'VMToHostButton'
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 130
 $System_Drawing_Size.Height = 20
 $VMToHostButton.Size = $System_Drawing_Size
 $VMToHostButton.Font = new-object System.Drawing.Font("Tahoma",8)
 $VMToHostButton.UseVisualStyleBackColor = $True
 
 $VMToHostButton.Text = 'VM to Host'
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 250
 $System_Drawing_Point.Y = 115
 $VMToHostButton.Location = $System_Drawing_Point
 $VMToHostButton.DataBindings.DefaultDataSourceUpdateMode = 0
 $VMToHostButton.add_Click($VMToHostButton_Click)
 $VMToHostButton.Enabled = $false
 
 $MigrationForm.Controls.Add($VMToHostButton)
 
 $DBtoDBButton.TabIndex = 3
 $DBtoDBButton.Name = 'DBtoDBButton'
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 130
 $System_Drawing_Size.Height = 20
 $DBtoDBButton.Size = $System_Drawing_Size
 $DBtoDBButton.Font = new-object System.Drawing.Font("Tahoma",8)
 $DBtoDBButton.UseVisualStyleBackColor = $True
 
 $DBtoDBButton.Text = 'Datastore to Datastore'
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 250
 $System_Drawing_Point.Y = 140
 $DBtoDBButton.Location = $System_Drawing_Point
 $DBtoDBButton.DataBindings.DefaultDataSourceUpdateMode = 0
 $DBtoDBButton.add_Click($DBtoDBButton_Click)
 $DBtoDBButton.Enabled = $false
 
 $MigrationForm.Controls.Add($DBtoDBButton)
 
 $VMtoDBButton.TabIndex = 3
 $VMtoDBButton.Name = 'VMtoDBButton'
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 130
 $System_Drawing_Size.Height = 20
 $VMtoDBButton.Size = $System_Drawing_Size
 $VMtoDBButton.Font = new-object System.Drawing.Font("Tahoma",8)
 $VMtoDBButton.UseVisualStyleBackColor = $True
 
 $VMtoDBButton.Text = 'VM to Datastore'
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 250
 $System_Drawing_Point.Y = 165
 $VMtoDBButton.Location = $System_Drawing_Point
 $VMtoDBButton.DataBindings.DefaultDataSourceUpdateMode = 0
 $VMtoDBButton.add_Click($VMtoDBButton_Click)
 $VMtoDBButton.Enabled = $false
 
 $MigrationForm.Controls.Add($VMtoDBButton)
 
 $SourceHostListBox.FormattingEnabled = $True
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 200
 $System_Drawing_Size.Height = 200
 $SourceHostListBox.Size = $System_Drawing_Size
 $SourceHostListBox.Font = new-object System.Drawing.Font("Tahoma",8)
 $SourceHostListBox.DataBindings.DefaultDataSourceUpdateMode = 0
 $SourceHostListBox.Name = 'SourceHostListBox'
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 30
 $System_Drawing_Point.Y = 210
 $SourceHostListBox.Location = $System_Drawing_Point
 $SourceHostListBox.MultiColumn = $false
 $SourceHostListBox.TabIndex = 5
 $SourceHostListBox.Sorted = $True
 
 $MigrationForm.Controls.Add($SourceHostListBox)
 
 $SourceDBListBox.FormattingEnabled = $True
 $SourceDBListBox.CheckOnClick = $True
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 200
 $System_Drawing_Size.Height = 200
 $SourceDBListBox.Size = $System_Drawing_Size
 $SourceDBListBox.Font = new-object System.Drawing.Font("Tahoma",8)
 $SourceDBListBox.DataBindings.DefaultDataSourceUpdateMode = 0
 $SourceDBListBox.Name = 'SourceDBListBox'
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 30
 $System_Drawing_Point.Y = 210
 $SourceDBListBox.Location = $System_Drawing_Point
 $SourceDBListBox.MultiColumn = $false
 $SourceDBListBox.TabIndex = 5
 $SourceDBListBox.Sorted = $True
 
 $SourceVMListBox.FormattingEnabled = $True
 $SourceVMListBox.CheckOnClick = $True
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 200
 $System_Drawing_Size.Height = 200
 $SourceVMListBox.Size = $System_Drawing_Size
 $SourceVMListBox.Font = new-object System.Drawing.Font("Tahoma",8)
 $SourceVMListBox.DataBindings.DefaultDataSourceUpdateMode = 0
 $SourceVMListBox.Name = 'SourceVMListBox'
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 30
 $System_Drawing_Point.Y = 210
 $SourceVMListBox.Location = $System_Drawing_Point
 $SourceVMListBox.MultiColumn = $false
 $SourceVMListBox.TabIndex = 5
 $SourceVMListBox.Sorted = $True
 
 $SourceButton.TabIndex = 7
 $SourceButton.Name = 'Source Button'
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 200
 $System_Drawing_Size.Height = 20
 $SourceButton.Size = $System_Drawing_Size
 $SourceButton.Font = new-object System.Drawing.Font("Tahoma",8)
 $SourceButton.UseVisualStyleBackColor = $True
 
 $SourceButton.Text = 'Source'
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 30
 $System_Drawing_Point.Y = 415
 $SourceButton.Location = $System_Drawing_Point
 $SourceButton.DataBindings.DefaultDataSourceUpdateMode = 0
 $SourceButton.add_Click($SourceButton_OnClick)
 $SourceButton.Enabled = $false
 
 $MigrationForm.Controls.Add($SourceButton)
 
 $DestinationListBox.FormattingEnabled = $True
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 200
 $System_Drawing_Size.Height = 200
 $DestinationListBox.Size = $System_Drawing_Size
 $DestinationListBox.Font = new-object System.Drawing.Font("Tahoma",8)
 $DestinationListBox.DataBindings.DefaultDataSourceUpdateMode = 0
 $DestinationListBox.Name = 'DestinationListBox'
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 250
 $System_Drawing_Point.Y = 210
 $DestinationListBox.Location = $System_Drawing_Point
 $DestinationListBox.MultiColumn = $false
 $DestinationListBox.TabIndex = 5
 $DestinationListBox.Sorted = $True
 
 $MigrationForm.Controls.Add($DestinationListBox)
 
 $DestinationButton.TabIndex = 7
 $DestinationButton.Name = 'Destination Button'
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 200
 $System_Drawing_Size.Height = 20
 $DestinationButton.Size = $System_Drawing_Size
 $DestinationButton.Font = new-object System.Drawing.Font("Tahoma",8)
 $DestinationButton.UseVisualStyleBackColor = $True
 
 $DestinationButton.Text = 'Destination'
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 250
 $System_Drawing_Point.Y = 415
 $DestinationButton.Location = $System_Drawing_Point
 $DestinationButton.DataBindings.DefaultDataSourceUpdateMode = 0
 $DestinationButton.add_Click($DestinationButton_OnClick)
 $DestinationButton.Enabled = $false
 
 $MigrationForm.Controls.Add($DestinationButton)
 
 $MigrateButton.TabIndex = 7
 $MigrateButton.Name = 'MigrateButton'
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 130
 $System_Drawing_Size.Height = 25
 $MigrateButton.Size = $System_Drawing_Size
 $MigrateButton.Font = new-object System.Drawing.Font("Tahoma",8)
 $MigrateButton.UseVisualStyleBackColor = $True
 
 $MigrateButton.Text = 'Migrate'
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 30
 $System_Drawing_Point.Y = 450
 $MigrateButton.Location = $System_Drawing_Point
 $MigrateButton.DataBindings.DefaultDataSourceUpdateMode = 0
 $MigrateButton.add_Click($MigrateButton_OnClick)
 $MigrateButton.Enabled = $false
 
 $MigrationForm.Controls.Add($MigrateButton)
 
 $ResetButton.TabIndex = 19
 $ResetButton.Name = 'ResetButton'
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 140
 $System_Drawing_Size.Height = 25
 $ResetButton.Size = $System_Drawing_Size
 $ResetButton.Font = new-object System.Drawing.Font("Tahoma",8)
 $ResetButton.UseVisualStyleBackColor = $True
 
 $ResetButton.Text = 'Reset'
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 170
 $System_Drawing_Point.Y = 450
 $ResetButton.Location = $System_Drawing_Point
 $ResetButton.DataBindings.DefaultDataSourceUpdateMode = 0
 $ResetButton.add_Click($ResetButton_OnClick)
 $ResetButton.Enabled = $false
 
 $MigrationForm.Controls.Add($ResetButton)
 
 $CloseButton.TabIndex = 19
 $CloseButton.Name = 'CloseButton'
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 130
 $System_Drawing_Size.Height = 25
 $CloseButton.Size = $System_Drawing_Size
 $CloseButton.Font = new-object System.Drawing.Font("Tahoma",8)
 $CloseButton.UseVisualStyleBackColor = $True
 
 $CloseButton.Text = 'Close'
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 320
 $System_Drawing_Point.Y = 450
 $CloseButton.Location = $System_Drawing_Point
 $CloseButton.DataBindings.DefaultDataSourceUpdateMode = 0
 $CloseButton.add_Click($CloseButton_OnClick)
 
 $MigrationForm.Controls.Add($CloseButton)
 
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 80
 $System_Drawing_Size.Height = 20
 $MessagesLabel.Size = $System_Drawing_Size
 $MessagesLabel.Font = new-object System.Drawing.Font("Tahoma",8,[System.Drawing.FontStyle]::Bold)
 $MessagesLabel.Text = 'Messages'
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 480
 $System_Drawing_Point.Y = 50
 $MessagesLabel.Location = $System_Drawing_Point
 $MessagesLabel.DataBindings.DefaultDataSourceUpdateMode = 0
 $MessagesLabel.Name = 'MessagesLabel'
 
 $MigrationForm.Controls.Add($MessagesLabel)
 
 $MessagesListBox.FormattingEnabled = $True
 $MessagesListBox.SelectionMode = "None"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 525
 $System_Drawing_Size.Height = 140
 $MessagesListBox.Size = $System_Drawing_Size
 $MessagesListBox.Font = new-object System.Drawing.Font("Tahoma",8)
 $MessagesListBox.DataBindings.DefaultDataSourceUpdateMode = 0
 $MessagesListBox.Name = 'MessagesListBox'
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 480
 $System_Drawing_Point.Y = 70
 $MessagesListBox.Location = $System_Drawing_Point
 $MessagesListBox.MultiColumn = $false
 $MessagesListBox.HorizontalScrollbar = $true
 
 $MigrationForm.Controls.Add($MessagesListBox)
 
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 80
 $System_Drawing_Size.Height = 20
 $TaskLabel.Size = $System_Drawing_Size
 $TaskLabel.Font = new-object System.Drawing.Font("Tahoma",8,[System.Drawing.FontStyle]::Bold)
 $TaskLabel.Text = 'Recent Tasks'
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 480
 $System_Drawing_Point.Y = 215
 $TaskLabel.Location = $System_Drawing_Point
 $TaskLabel.DataBindings.DefaultDataSourceUpdateMode = 0
 $TaskLabel.Name = 'TaskLabel'
 
 $MigrationForm.Controls.Add($TaskLabel)
 
 $TaskListBox.FormattingEnabled = $True
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 525
 $System_Drawing_Size.Height = 200
 $TaskListBox.Size = $System_Drawing_Size
 $TaskListBox.Font = new-object System.Drawing.Font("Tahoma",8)
 $TaskListBox.DataBindings.DefaultDataSourceUpdateMode = 0
 $TaskListBox.Name = 'TaskListBox'
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 480
 $System_Drawing_Point.Y = 235
 $TaskListBox.Location = $System_Drawing_Point
 $TaskListBox.MultiColumn = $false
 $TaskListBox.HorizontalScrollbar = $true
 $TaskListBox.SelectionMode = "MultiExtended"
 
 $MigrationForm.Controls.Add($TaskListBox)
 
 $ClearMsgButton.TabIndex = 0
 $ClearMsgButton.Name = 'ClearMsgButton'
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 120
 $System_Drawing_Size.Height = 25
 $ClearMsgButton.Size = $System_Drawing_Size
 $ClearMsgButton.Font = new-object System.Drawing.Font("Tahoma",8)
 $ClearMsgButton.UseVisualStyleBackColor = $True
 
 $ClearMsgButton.Text = 'Clear Messages'
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 480
 $System_Drawing_Point.Y = 450
 $ClearMsgButton.Location = $System_Drawing_Point
 $ClearMsgButton.DataBindings.DefaultDataSourceUpdateMode = 0
 $ClearMsgButton.add_Click($ClearMsgButton_Click)
 
 $MigrationForm.Controls.Add($ClearMsgButton)
 
 $UndoButton.TabIndex = 0
 $UndoButton.Name = 'UndoButton'
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 120
 $System_Drawing_Size.Height = 25
 $UndoButton.Size = $System_Drawing_Size
 $UndoButton.Font = new-object System.Drawing.Font("Tahoma",8)
 $UndoButton.UseVisualStyleBackColor = $True
 
 $UndoButton.Text = 'Undo'
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 615
 $System_Drawing_Point.Y = 450
 $UndoButton.Location = $System_Drawing_Point
 $UndoButton.DataBindings.DefaultDataSourceUpdateMode = 0
 $UndoButton.add_Click($UndoButton_Click)
 
 $MigrationForm.Controls.Add($UndoButton)
 
 $UndoALLButton.TabIndex = 0
 $UndoALLButton.Name = 'UndoALLButton'
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 120
 $System_Drawing_Size.Height = 25
 $UndoALLButton.Size = $System_Drawing_Size
 $UndoALLButton.Font = new-object System.Drawing.Font("Tahoma",8)
 $UndoALLButton.UseVisualStyleBackColor = $True
 
 $UndoALLButton.Text = 'Undo ALL'
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 750
 $System_Drawing_Point.Y = 450
 $UndoALLButton.Location = $System_Drawing_Point
 $UndoALLButton.DataBindings.DefaultDataSourceUpdateMode = 0
 $UndoALLButton.add_Click($UndoALLButton_Click)
 
 $MigrationForm.Controls.Add($UndoALLButton)
 
 $SaveTaskButton.TabIndex = 0
 $SaveTaskButton.Name = 'SaveTaskButton'
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 120
 $System_Drawing_Size.Height = 25
 $SaveTaskButton.Size = $System_Drawing_Size
 $SaveTaskButton.Font = new-object System.Drawing.Font("Tahoma",8)
 $SaveTaskButton.UseVisualStyleBackColor = $True
 
 $SaveTaskButton.Text = 'Save Tasks'
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 885
 $System_Drawing_Point.Y = 450
 $SaveTaskButton.Location = $System_Drawing_Point
 $SaveTaskButton.DataBindings.DefaultDataSourceUpdateMode = 0
 $SaveTaskButton.add_Click($SaveTaskButton_Click)
 
 $MigrationForm.Controls.Add($SaveTaskButton)
 
 
 $InitialFormWindowState = $MigrationForm.WindowState
 $MigrationForm.add_Load($OnLoadForm_StateCorrection)
 $MigrationForm.ShowDialog()| Out-Null
 
 
 GenerateForm
`

