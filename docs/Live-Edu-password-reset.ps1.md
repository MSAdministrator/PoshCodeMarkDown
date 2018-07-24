---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2411
Published Date: 
Archived Date: 2010-12-24t06
---

# live@edu password reset - 

## Description

simple windows form to connect to live@edu for the purpose of searching for accounts and resetting their passwords.

## Comments



## Usage



## TODO



## function

`connect-eduservice`

## Code

`#
 #
 #----------------------------------------------
 #----------------------------------------------
 
 function OnApplicationLoad {
 	
 }
 
 function OnApplicationExit {
 	
 	Get-PSSession | ? { $_.ConfigurationName -match 'Exchange' } | Remove-PSSession
 	
 }
 
 
 #----------------------------------------------
 #----------------------------------------------
 function GenerateForm {
 
 	#----------------------------------------------
 	#----------------------------------------------
 	[void][reflection.assembly]::Load("System.Windows.Forms, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089")
 	[void][reflection.assembly]::Load("System.Drawing, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a")
 	[void][reflection.assembly]::Load("mscorlib, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089")
 	[void][reflection.assembly]::Load("System, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089")
 	
 	#----------------------------------------------
 	#----------------------------------------------
 	[System.Windows.Forms.Application]::EnableVisualStyles()
 	$form1 = New-Object System.Windows.Forms.Form
 	$lblLiveEDUStatus = New-Object System.Windows.Forms.Label
 	$grpPassword = New-Object System.Windows.Forms.GroupBox
 	$lblConfirmPass = New-Object System.Windows.Forms.Label
 	$lblNewPass = New-Object System.Windows.Forms.Label
 	$btnCancel = New-Object System.Windows.Forms.Button
 	$btnOK = New-Object System.Windows.Forms.Button
 	$txtConfirmPass = New-Object System.Windows.Forms.TextBox
 	$txtNewPass = New-Object System.Windows.Forms.TextBox
 	$gbxSearch = New-Object System.Windows.Forms.GroupBox
 	$lblName = New-Object System.Windows.Forms.Label
 	$txtName = New-Object System.Windows.Forms.TextBox
 	$btnSearch = New-Object System.Windows.Forms.Button
 	$lstResults = New-Object System.Windows.Forms.ListView
 	$StudentName = New-Object System.Windows.Forms.ColumnHeader
 	$StudentEmail = New-Object System.Windows.Forms.ColumnHeader
 	$InitialFormWindowState = New-Object System.Windows.Forms.FormWindowState
 
 	#----------------------------------------------
 	#----------------------------------------------
 
 	
 	
 	
 	
 	$FormEvent_Load={
 		function Connect-EDUService
 		{
 			param
 			(
 				$username,
 				$password
 			)
 		    $success = $false
 		    try
 		    {
 		    	[string] $RemoteURL = "https://ps.outlook.com/powershell"
 		    	if ($username -eq $null -and $password -eq $null)
 				{
 					$livecred = get-credential
 				}
 				else
 				{
 					$password = ConvertTo-SecureString $password -AsPlainText -Force
 					$livecred = New-Object System.Management.Automation.PSCredential $username, $password
 				}
 		    	$eduSession = New-PSSession -ConfigurationName microsoft.exchange `
 		    	                            -ConnectionUri $RemoteURL `
 		    	                            -Credential $liveCred `
 		    	                            -Authentication Basic -AllowRedirection -ErrorAction Stop
 		    	Import-PSSession $eduSession -prefix "EDU" | Out-Null
 		      $success = $true
 		    }
 		    catch
 		    {
 		    }
 		    $success
 		}
 		$eduConnected = Connect-EDUService
 		
 		if ($eduConnected)
 		{
 			$lblLiveEDUStatus.BackColor = [Drawing.Color]::Green
 			$lblLiveEDUStatus.ForeColor = [Drawing.Color]::White
 			$lblLiveEDUStatus.Text = "Successfully connected to Live@EDU Servers"
 		}
 		else
 		{
 			$lblLiveEDUStatus.BackColor = [Drawing.Color]::Red
 			$lblLiveEDUStatus.ForeColor = [Drawing.Color]::White
 			$lblLiveEDUStatus.Text = "Live@EDU Connection FAILED!"
 		}	
 	}
 	
 	$handler_btnOK_Click={
 		$passwordsMatch = $false
 		if ($txtNewPass.Text -ceq $txtConfirmPass.Text)
 		{
 			if ($txtNewPass.Text.Length -ge 6)
 			{
 				$newPass = ConvertTo-SecureString $txtNewPass.Text -AsPlainText -Force
 				$passwordsMatch = $true
 			}
 			else
 			{
 				$message = "Error: New password must be 6 characters or longer."
 				$caption = "Error"
 				$buttons = [System.Windows.Forms.MessageBoxButtons]::OK
 				$icon = [System.Windows.Forms.MessageBoxIcon]::Error
 				[System.Windows.Forms.MessageBox]::Show($message,$caption,$buttons,$icon)			
 			}
 		}
 		else
 		{
 			$message = "Error: Passwords do not match."
 			$caption = "Error"
 			$buttons = [System.Windows.Forms.MessageBoxButtons]::OK
 			$icon = [System.Windows.Forms.MessageBoxIcon]::Error
 			[System.Windows.Forms.MessageBox]::Show($message,$caption,$buttons,$icon)
 		}
 		
 		if ($lstResults.SelectedItems.Count -gt 0 -and $passwordsMatch)
 		{
 			foreach ($i in $lstResults.SelectedItems)
 			{
 				$error.clear()
 				Set-EDUMailbox -Identity $i.subitems[1].text -Password $newPass
 				if ($error[0] -ne $null)
 				{
 					$message = $error[0].Exception.Message
 					$caption = "Error"
 					$buttons = [System.Windows.Forms.MessageBoxButtons]::OK
 					$icon = [System.Windows.Forms.MessageBoxIcon]::Error
 					[System.Windows.Forms.MessageBox]::Show($message,$caption,$buttons,$icon)
 				}
 				else
 				{
 					$message = "Password for {0} successfully changed" -f $i.Text
 					$caption = "Success"
 					$buttons = [System.Windows.Forms.MessageBoxButtons]::OK
 					$icon = [System.Windows.Forms.MessageBoxIcon]::Information
 					[System.Windows.Forms.MessageBox]::Show($message,$caption,$buttons,$icon)
 				}
 			}
 		}
 		elseif ($passwordsMatch)
 		{
 			$message = "Error: You must have a student selected."
 			$caption = "Error"
 			$buttons = [System.Windows.Forms.MessageBoxButtons]::OK
 			$icon = [System.Windows.Forms.MessageBoxIcon]::Error
 			[System.Windows.Forms.MessageBox]::Show($message,$caption,$buttons,$icon)
 		}
 	}
 	
 	$handler_btnCancel_Click={
 		$form1.Close()
 	}
 	
 	$handler_btnSearch_Click={
 		$searchResults = @()
 		Get-EduMailBox -anr $txtName.Text | 
 		 ForEach-Object { $searchResults += $_ }
 		
 		$lstResults.BeginUpdate()
 		$lstResults.SelectedItems.Clear()
 		$lstResults.Items.Clear()
 		if ($searchResults.count -gt 0)
 		{
 			foreach ($r in $searchResults)
 			{
 				$lvi = New-Object System.Windows.Forms.ListViewItem
 				$lvi.Text = $r.displayname
 				$liveID = $r.windowsLiveId
 				foreach ($si in $liveID)
 				{
 					if ($si -ne $null)
 					{
 						$lvi.SubItems.Add($si)
 					}
 					else
 					{
 						$lvi.SubItems.Add("")
 					}
 				}
 				$lstResults.Items.Add($lvi)
 			}
 		}
 		$lstResults.EndUpdate()
 	}
 	
 	$handler_lstResults_SelectedIndexChanged={
 		if ($lstResults.SelectedItems.Count -gt 0)
 		{
 			$grpPassword.Enabled = $true
 		}
 		else
 		{
 			$grpPassword.Enabled = $false
 		}
 	}
 	
 	
 	#----------------------------------------------
 	#----------------------------------------------
 	
 	$Form_StateCorrection_Load=
 	{
 		$form1.WindowState = $InitialFormWindowState
 	}
 	
 	#----------------------------------------------
 	#----------------------------------------------
 	#
 	#
 	$form1.Controls.Add($lblLiveEDUStatus)
 	$form1.Controls.Add($grpPassword)
 	$form1.Controls.Add($gbxSearch)
 	$form1.Controls.Add($lstResults)
 	$form1.ClientSize = New-Object System.Drawing.Size(659,339)
 	$form1.DataBindings.DefaultDataSourceUpdateMode = [System.Windows.Forms.DataSourceUpdateMode]::OnValidation 
 	$form1.Name = "form1"
 	$form1.Text = "Student Email Password Reset"
 	$form1.add_Load($FormEvent_Load)
 	#
 	#
 	$lblLiveEDUStatus.Anchor = [System.Windows.Forms.AnchorStyles]::Bottom -bor [System.Windows.Forms.AnchorStyles]::Left 
 	$lblLiveEDUStatus.DataBindings.DefaultDataSourceUpdateMode = [System.Windows.Forms.DataSourceUpdateMode]::OnValidation 
 	$lblLiveEDUStatus.Location = New-Object System.Drawing.Point(0,317)
 	$lblLiveEDUStatus.Name = "lblLiveEDUStatus"
 	$lblLiveEDUStatus.Size = New-Object System.Drawing.Size(331,21)
 	$lblLiveEDUStatus.TabIndex = 14
 	$lblLiveEDUStatus.TextAlign = [System.Drawing.ContentAlignment]::MiddleLeft 
 	#
 	#
 	$grpPassword.Controls.Add($lblConfirmPass)
 	$grpPassword.Controls.Add($lblNewPass)
 	$grpPassword.Controls.Add($btnCancel)
 	$grpPassword.Controls.Add($btnOK)
 	$grpPassword.Controls.Add($txtConfirmPass)
 	$grpPassword.Controls.Add($txtNewPass)
 	$grpPassword.DataBindings.DefaultDataSourceUpdateMode = [System.Windows.Forms.DataSourceUpdateMode]::OnValidation 
 	$grpPassword.Enabled = $False
 	$grpPassword.Location = New-Object System.Drawing.Point(338,13)
 	$grpPassword.Name = "grpPassword"
 	$grpPassword.Size = New-Object System.Drawing.Size(309,99)
 	$grpPassword.TabIndex = 12
 	$grpPassword.TabStop = $False
 	$grpPassword.Text = "Change Password"
 	#
 	#
 	$lblConfirmPass.DataBindings.DefaultDataSourceUpdateMode = [System.Windows.Forms.DataSourceUpdateMode]::OnValidation 
 	$lblConfirmPass.Location = New-Object System.Drawing.Point(6,41)
 	$lblConfirmPass.Name = "lblConfirmPass"
 	$lblConfirmPass.Size = New-Object System.Drawing.Size(86,23)
 	$lblConfirmPass.TabIndex = 13
 	$lblConfirmPass.Text = "Confirm: "
 	$lblConfirmPass.TextAlign = [System.Drawing.ContentAlignment]::MiddleRight 
 	#
 	#
 	$lblNewPass.DataBindings.DefaultDataSourceUpdateMode = [System.Windows.Forms.DataSourceUpdateMode]::OnValidation 
 	$lblNewPass.Location = New-Object System.Drawing.Point(6,15)
 	$lblNewPass.Name = "lblNewPass"
 	$lblNewPass.Size = New-Object System.Drawing.Size(86,23)
 	$lblNewPass.TabIndex = 12
 	$lblNewPass.Text = "New Password: "
 	$lblNewPass.TextAlign = [System.Drawing.ContentAlignment]::MiddleRight 
 	#
 	#
 	$btnCancel.DataBindings.DefaultDataSourceUpdateMode = [System.Windows.Forms.DataSourceUpdateMode]::OnValidation 
 	$btnCancel.Location = New-Object System.Drawing.Point(227,67)
 	$btnCancel.Name = "btnCancel"
 	$btnCancel.Size = New-Object System.Drawing.Size(75,23)
 	$btnCancel.TabIndex = 6
 	$btnCancel.Text = "Cancel"
 	$btnCancel.UseVisualStyleBackColor = $True
 	$btnCancel.add_Click($handler_btnCancel_Click)
 	#
 	#
 	$btnOK.DataBindings.DefaultDataSourceUpdateMode = [System.Windows.Forms.DataSourceUpdateMode]::OnValidation 
 	$btnOK.Location = New-Object System.Drawing.Point(146,67)
 	$btnOK.Name = "btnOK"
 	$btnOK.Size = New-Object System.Drawing.Size(75,23)
 	$btnOK.TabIndex = 5
 	$btnOK.Text = "OK"
 	$btnOK.UseVisualStyleBackColor = $True
 	$btnOK.add_Click($handler_btnOK_Click)
 	#
 	#
 	$txtConfirmPass.DataBindings.DefaultDataSourceUpdateMode = [System.Windows.Forms.DataSourceUpdateMode]::OnValidation 
 	$txtConfirmPass.Location = New-Object System.Drawing.Point(98,41)
 	$txtConfirmPass.Name = "txtConfirmPass"
 	$txtConfirmPass.Size = New-Object System.Drawing.Size(204,20)
 	$txtConfirmPass.TabIndex = 4
 	#
 	#
 	$txtNewPass.DataBindings.DefaultDataSourceUpdateMode = [System.Windows.Forms.DataSourceUpdateMode]::OnValidation 
 	$txtNewPass.Location = New-Object System.Drawing.Point(98,15)
 	$txtNewPass.Name = "txtNewPass"
 	$txtNewPass.Size = New-Object System.Drawing.Size(204,20)
 	$txtNewPass.TabIndex = 3
 	#
 	#
 	$gbxSearch.Controls.Add($lblName)
 	$gbxSearch.Controls.Add($txtName)
 	$gbxSearch.Controls.Add($btnSearch)
 	$gbxSearch.DataBindings.DefaultDataSourceUpdateMode = [System.Windows.Forms.DataSourceUpdateMode]::OnValidation 
 	$gbxSearch.Location = New-Object System.Drawing.Point(13,12)
 	$gbxSearch.Name = "gbxSearch"
 	$gbxSearch.Size = New-Object System.Drawing.Size(318,100)
 	$gbxSearch.TabIndex = 11
 	$gbxSearch.TabStop = $False
 	$gbxSearch.Text = "Student Search"
 	#
 	#
 	$lblName.DataBindings.DefaultDataSourceUpdateMode = [System.Windows.Forms.DataSourceUpdateMode]::OnValidation 
 	$lblName.Location = New-Object System.Drawing.Point(6,16)
 	$lblName.Name = "lblName"
 	$lblName.Size = New-Object System.Drawing.Size(88,23)
 	$lblName.TabIndex = 4
 	$lblName.Text = "Student Name: "
 	$lblName.TextAlign = [System.Drawing.ContentAlignment]::MiddleRight 
 	#
 	#
 	$txtName.DataBindings.DefaultDataSourceUpdateMode = [System.Windows.Forms.DataSourceUpdateMode]::OnValidation 
 	$txtName.Location = New-Object System.Drawing.Point(100,16)
 	$txtName.Name = "txtName"
 	$txtName.Size = New-Object System.Drawing.Size(212,20)
 	$txtName.TabIndex = 0
 	#
 	#
 	$btnSearch.DataBindings.DefaultDataSourceUpdateMode = [System.Windows.Forms.DataSourceUpdateMode]::OnValidation 
 	$btnSearch.Location = New-Object System.Drawing.Point(237,42)
 	$btnSearch.Name = "btnSearch"
 	$btnSearch.Size = New-Object System.Drawing.Size(75,23)
 	$btnSearch.TabIndex = 1
 	$btnSearch.Text = "Search"
 	$btnSearch.UseVisualStyleBackColor = $True
 	$btnSearch.add_Click($handler_btnSearch_Click)
 	#
 	#
 	$lstResults.Anchor = [System.Windows.Forms.AnchorStyles]::Top -bor [System.Windows.Forms.AnchorStyles]::Bottom -bor [System.Windows.Forms.AnchorStyles]::Left -bor [System.Windows.Forms.AnchorStyles]::Right 
 	$System_Windows_Forms_ColumnHeader_11 = New-Object System.Windows.Forms.ColumnHeader
 	$System_Windows_Forms_ColumnHeader_11.Text = "Student Name"
 	$System_Windows_Forms_ColumnHeader_11.Width = 157
 
 	[void]$lstResults.Columns.Add($System_Windows_Forms_ColumnHeader_11)
 	$System_Windows_Forms_ColumnHeader_12 = New-Object System.Windows.Forms.ColumnHeader
 	$System_Windows_Forms_ColumnHeader_12.Text = "Email Address"
 	$System_Windows_Forms_ColumnHeader_12.Width = 218
 
 	[void]$lstResults.Columns.Add($System_Windows_Forms_ColumnHeader_12)
 	$lstResults.DataBindings.DefaultDataSourceUpdateMode = [System.Windows.Forms.DataSourceUpdateMode]::OnValidation 
 	$lstResults.FullRowSelect = $True
 	$lstResults.GridLines = $True
 	$lstResults.HeaderStyle = [System.Windows.Forms.ColumnHeaderStyle]::Nonclickable 
 	$lstResults.HideSelection = $False
 	$lstResults.Location = New-Object System.Drawing.Point(13,118)
 	$lstResults.MultiSelect = $False
 	$lstResults.Name = "lstResults"
 	$lstResults.Size = New-Object System.Drawing.Size(634,193)
 	$lstResults.TabIndex = 2
 	$lstResults.UseCompatibleStateImageBehavior = $False
 	$lstResults.View = [System.Windows.Forms.View]::Details 
 	$lstResults.add_SelectedIndexChanged($handler_lstResults_SelectedIndexChanged)
 	#
 	#
 	$StudentName.Text = "Student Name"
 	$StudentName.Width = 157
 	#
 	#
 	$StudentEmail.Text = "Email Address"
 	$StudentEmail.Width = 218
 
 	#----------------------------------------------
 
 	$InitialFormWindowState = $form1.WindowState
 	$form1.add_Load($Form_StateCorrection_Load)
 	return $form1.ShowDialog()
 
 
 if(OnApplicationLoad -eq $true)
 {
 	GenerateForm | Out-Null
 	OnApplicationExit
 }
`

