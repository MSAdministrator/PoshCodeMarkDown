---
Author: david chung
Publisher: 
Copyright: 
Email: 
Version: 1.4
Encoding: ascii
License: cc0
PoshCode ID: 3202
Published Date: 2012-02-04t15
Archived Date: 2012-02-07t03
---

# hostprofile update gui - 

## Description

gui interface that helps you update host profiles faster.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ################################################################################################################
 #
 #
 #
 ################################################################################################################
 
 function GenerateForm {
 
 [reflection.assembly]::loadwithpartialname("System.Drawing") | Out-Null
 [reflection.assembly]::loadwithpartialname("System.Windows.Forms") | Out-Null
 
 $form1 = New-Object System.Windows.Forms.Form
 $button4 = New-Object System.Windows.Forms.Button
 $groupBox3 = New-Object System.Windows.Forms.GroupBox
 $ComplianceBox1 = New-Object System.Windows.Forms.CheckBox
 $label4 = New-Object System.Windows.Forms.Label
 $ClusterName = New-Object System.Windows.Forms.ListBox
 $BeginButton = New-Object System.Windows.Forms.Button
 $CloseButton = New-Object System.Windows.Forms.Button
 $groupBox2 = New-Object System.Windows.Forms.GroupBox
 $ProgressLog = New-Object System.Windows.Forms.ListBox
 $groupBox1 = New-Object System.Windows.Forms.GroupBox
 $LoginButton = New-Object System.Windows.Forms.Button
 $Password = New-Object System.Windows.Forms.TextBox
 $User = New-Object System.Windows.Forms.TextBox
 $vCenter = New-Object System.Windows.Forms.TextBox
 $label3 = New-Object System.Windows.Forms.Label
 $label2 = New-Object System.Windows.Forms.Label
 $label1 = New-Object System.Windows.Forms.Label
 $InitialFormWindowState = New-Object System.Windows.Forms.FormWindowState
 
 #----------------------------------------------
 #----------------------------------------------
 $BeginButton_OnClick= 
 {
 $scluster = $ClusterName.SelectedItem
 if ($scluster -eq $null) {
 	$ProgressLog.items.add("")
 	$ProgressLog.items.add("Please select a cluster from the cluster list")
 	[Windows.Forms.Application]::DoEvents()	
 	}
 Else {
 	$vcluster = Get-Cluster $scluster
 	$ProgressLog.items.add("Collecting Powered On Hosts in $vcluster...")	
 	[Windows.Forms.Application]::DoEvents()	
 	$VMhosts = $vcluster| Get-VMHost | where {$_.PowerState -eq "PoweredOn"} | sort name
 	$ProgressLog.items.add("....completed")	
 	$ProgressLog.items.add("")
 	[Windows.Forms.Application]::DoEvents()
 	If ($ComplianceBox1.Checked) {
 		$ProgressLog.items.add("Checking Hosts in $vcluster...")
 		[Windows.Forms.Application]::DoEvents()
 		ForEach ($VMhost in $VMhosts) {
 			$profile = Test-VMHostProfileCompliance -VMHost $VMhost
 			If ($PROFILE -ne $null) {
 				$hostprofile = Get-vmhostprofile -entity $VMhost
 				$ProgressLog.items.add("$VMHost is NOT Compliant.")
 				[Windows.Forms.Application]::DoEvents()
 			}
 			Else {
 				$ProgressLog.items.add("$vmhost is Compliant.")
 				[Windows.Forms.Application]::DoEvents()
 			}
 		}
 	$ProgressLog.items.add("")
 	$ProgressLog.items.add("Compliance Check is Completed")	
 	$ProgressLog.items.add("")
 	}
 	Else {
 		$ProgressLog.items.add("Checking Hosts in $vcluster...")
 		[Windows.Forms.Application]::DoEvents()
 		$VMhosts = $vcluster| Get-VMHost | where {$_.PowerState -eq "PoweredOn"} | sort name
 		$ClusterHostCount = @($VMhosts).Count
 	    $ClusterConnectedCount = @($VMhosts | where {$_.ConnectionState -eq "Connected"}).Count
 		If ($ClusterHostCount -ne $ClusterConnectedCount){
 	        $ProgressLog.items.add("WARNING: One or more hosts in $vcluster in Maint. mode.")
 			$ProgressLog.items.add("Please try to remediate before updating the host profile.")
 	       	[Windows.Forms.Application]::DoEvents()
 			}
 		Else {
 			ForEach ($VMhost in $VMhosts) {
 				$profile = Test-VMHostProfileCompliance -VMHost $VMhost
 				If ($PROFILE -ne $null) {
 					$hostprofile = Get-vmhostprofile -entity $VMhost
 					$ProgressLog.items.add("$VMHost is NOT Compliant.")
 					[Windows.Forms.Application]::DoEvents()
 					if ($vcluster.DrsAutomationLevel -ne "FullyAutomated"){
 				    	$ProgressLog.items.add("WARNING: DRS FullyAutomated not set on $vcluster")
 						$ProgressLog.items.add("WARNING: Enabling DRS FullyAutomated")
 						[Windows.Forms.Application]::DoEvents()
 			       		Set-Cluster -Cluster $vcluster -DrsAutomationLevel FullyAutomated -Confirm:$false
 			    	}
 			    	foreach ($vm in ($VMHost | get-vm)){
 			        	if ((get-cddrive -VM $vm).ConnectionState.Connected -eq "true"){
 			            	$ProgressLog.items.add("WARNING: $VM has a CDROM drive attached, detaching.")
 							$ProgressLog.items.add("WARNING: Detaching CDROM.")
 							[Windows.Forms.Application]::DoEvents()
 			            	Set-CDDrive -CD (get-cddrive -VM $vm) -StartConnected:$false -Connected:$False -Confirm:$false 
 			        	}
 					}
 					$ProgressLog.items.add("Setting $VMhost to Maintenance Mode...")
 					[Windows.Forms.Application]::DoEvents()
 					Set-VMHost -State 'Maintenance' -VMHost $VMhost -Evacuate:$true -Confirm:$false
 					$ProgressLog.items.add("Applying Host Profilie...")
 					[Windows.Forms.Application]::DoEvents()
 					Apply-VMHostProfile -Entity $VMHost -Profile $hostprofile -ErrorAction SilentlyContinue -Confirm:$false
 					$ProgressLog.items.add("Taking the Host out of maintenance mode....")
 					[Windows.Forms.Application]::DoEvents()
 			    	Set-VMHost -VMHost $VMhost -State 'Connected'
 					Test-VMHostProfileCompliance -VMHost $VMhost 
 				}	
 				Else {
 					$ProgressLog.items.add("$vmhost is Compliant.")
 					$ProgressLog.items.add("")
 					[Windows.Forms.Application]::DoEvents()
 				}
 			}
 			$ProgressLog.items.add("Compliance Update is Completed")
 			}
 		}
 	}	
 }
 
 $CloseButton_OnClick= 
 {
 $form1.Close()
 }
 
 $LoginButton_OnClick= 
 {
 if ($VCenter.text -like "" -or $Password.text -like "" -or $User.text -like "") {
 	$ProgressLog.items.add("Field can not be empty.  Try Again")
 	[Windows.Forms.Application]::DoEvents()
 	}
 Else {	
 	$LoginButton.Enabled = $false
 	$V = $vCenter.text
 	$PSW = $Password.text
 	$iUser = $User.text
 	#$BSTR = [System.Runtime.InteropServices.marshal]::SecureStringToBSTR($PSW)
 	#$PSW = [System.Runtime.InteropServices.marshal]::PtrToStringAuto($BSTR)
 	$ProgressLog.items.add("Connecting to $V...")
 	[Windows.Forms.Application]::DoEvents()
 	$connection = Connect-VIServer $V -User $iuser -Password $PSW -WarningAction:SilentlyContinue
 	if ($connection.Port -notlike "*443*") {
 		$ProgressLog.items.add("....Failed!")
 		$ProgressLog.items.add("Failed to connect. Check vCenter and credentials.")
 		$LoginButton.Enabled = $true
 		[Windows.Forms.Application]::DoEvents()
 	}
 	Else {
 	$ProgressLog.items.add("....connected")
 	$ProgressLog.items.add("")
 	$ProgressLog.items.add("Getting Cluster Name from $V...")
 	[Windows.Forms.Application]::DoEvents()
 	$Clusters = Get-Cluster | sort name
 		Foreach ($Cluster in $Clusters){
 			$ClusterName.items.add($Cluster.Name)
 			[Windows.Forms.Application]::DoEvents()
 		}
 	$ProgressLog.items.add("....completed")	
 	[Windows.Forms.Application]::DoEvents()	
 	$BeginButton.Enabled = $True
 	}
 }	
 }
 $button4_OnClick= 
 {
 $vCenter.Text = $null
 $User.Text = $null
 $Password.Text = $null
 $ClusterName.items.clear()
 $ProgressLog.items.clear()
 [Windows.Forms.Application]::DoEvents()
 $BeginButton.Enabled = $false
 $LoginButton.Enabled = $true
 Disconnect-VIServer * -ErrorAction:SilentlyContinue -Confirm:$false 
 }
 
 
 
 $OnLoadForm_StateCorrection=
 	$form1.WindowState = $InitialFormWindowState
 }
 
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 583
 $System_Drawing_Size.Width = 452
 $form1.ClientSize = $System_Drawing_Size
 $form1.DataBindings.DefaultDataSourceUpdateMode = 0
 $form1.FormBorderStyle = 1
 $form1.Name = "form1"
 $form1.Text = "Host Profile Update by David Chung"
 
 
 $button4.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 277
 $System_Drawing_Point.Y = 548
 $button4.Location = $System_Drawing_Point
 $button4.Name = "button4"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 23
 $System_Drawing_Size.Width = 75
 $button4.Size = $System_Drawing_Size
 $button4.TabIndex = 5
 $button4.Text = "Reset Form"
 $button4.UseVisualStyleBackColor = $True
 $button4.add_Click($button4_OnClick)
 
 $form1.Controls.Add($button4)
 
 
 $groupBox3.DataBindings.DefaultDataSourceUpdateMode = 0
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 13
 $System_Drawing_Point.Y = 124
 $groupBox3.Location = $System_Drawing_Point
 $groupBox3.Name = "groupBox3"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 66
 $System_Drawing_Size.Width = 427
 $groupBox3.Size = $System_Drawing_Size
 $groupBox3.TabIndex = 4
 $groupBox3.TabStop = $False
 $groupBox3.Text = "Profile Update"
 
 $form1.Controls.Add($groupBox3)
 
 $ComplianceBox1.CheckAlign = 64
 $ComplianceBox1.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 262
 $System_Drawing_Point.Y = 42
 $ComplianceBox1.Location = $System_Drawing_Point
 $ComplianceBox1.Name = "ComplianceBox1"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 18
 $System_Drawing_Size.Width = 150
 $ComplianceBox1.Size = $System_Drawing_Size
 $ComplianceBox1.TabIndex = 3
 $ComplianceBox1.Text = "Compliance Check Only"
 $ComplianceBox1.UseVisualStyleBackColor = $True
 
 $groupBox3.Controls.Add($ComplianceBox1)
 
 $label4.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 7
 $System_Drawing_Point.Y = 29
 $label4.Location = $System_Drawing_Point
 $label4.Name = "label4"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 23
 $System_Drawing_Size.Width = 100
 $label4.Size = $System_Drawing_Size
 $label4.TabIndex = 2
 $label4.Text = "Select Cluster"
 
 $groupBox3.Controls.Add($label4)
 
 $ClusterName.DataBindings.DefaultDataSourceUpdateMode = 0
 $ClusterName.FormattingEnabled = $True
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 113
 $System_Drawing_Point.Y = 19
 $ClusterName.Location = $System_Drawing_Point
 $ClusterName.Name = "ClusterName"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 30
 $System_Drawing_Size.Width = 122
 $ClusterName.Size = $System_Drawing_Size
 $ClusterName.Sorted = $True
 $ClusterName.TabIndex = 0
 
 $groupBox3.Controls.Add($ClusterName)
 
 
 $BeginButton.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 345
 $System_Drawing_Point.Y = 13
 $BeginButton.Location = $System_Drawing_Point
 $BeginButton.Name = "BeginButton"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 23
 $System_Drawing_Size.Width = 75
 $BeginButton.Size = $System_Drawing_Size
 $BeginButton.TabIndex = 2
 $BeginButton.Text = "Begin"
 $BeginButton.UseVisualStyleBackColor = $True
 $BeginButton.add_Click($BeginButton_OnClick)
 $BeginButton.Enabled = $False
 #
 
 $groupBox3.Controls.Add($BeginButton)
 
 
 
 $CloseButton.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 358
 $System_Drawing_Point.Y = 548
 $CloseButton.Location = $System_Drawing_Point
 $CloseButton.Name = "CloseButton"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 23
 $System_Drawing_Size.Width = 75
 $CloseButton.Size = $System_Drawing_Size
 $CloseButton.TabIndex = 3
 $CloseButton.Text = "Close"
 $CloseButton.UseVisualStyleBackColor = $True
 $CloseButton.add_Click($CloseButton_OnClick)
 
 $form1.Controls.Add($CloseButton)
 
 
 $groupBox2.DataBindings.DefaultDataSourceUpdateMode = 0
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 13
 $System_Drawing_Point.Y = 196
 $groupBox2.Location = $System_Drawing_Point
 $groupBox2.Name = "groupBox2"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 345
 $System_Drawing_Size.Width = 427
 $groupBox2.Size = $System_Drawing_Size
 $groupBox2.TabIndex = 1
 $groupBox2.TabStop = $False
 $groupBox2.Text = "Progress Log"
 
 $form1.Controls.Add($groupBox2)
 $ProgressLog.DataBindings.DefaultDataSourceUpdateMode = 0
 $ProgressLog.FormattingEnabled = $True
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 6
 $System_Drawing_Point.Y = 20
 $ProgressLog.Location = $System_Drawing_Point
 $ProgressLog.Name = "ProgressLog"
 $ProgressLog.SelectionMode = 0
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 316
 $System_Drawing_Size.Width = 414
 $ProgressLog.Size = $System_Drawing_Size
 $ProgressLog.TabIndex = 3
 
 $groupBox2.Controls.Add($ProgressLog)
 
 
 
 $groupBox1.DataBindings.DefaultDataSourceUpdateMode = 0
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 13
 $System_Drawing_Point.Y = 12
 $groupBox1.Location = $System_Drawing_Point
 $groupBox1.Name = "groupBox1"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 106
 $System_Drawing_Size.Width = 427
 $groupBox1.Size = $System_Drawing_Size
 $groupBox1.TabIndex = 0
 $groupBox1.TabStop = $False
 $groupBox1.Text = "vCenter Login"
 
 $form1.Controls.Add($groupBox1)
 
 $LoginButton.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 345
 $System_Drawing_Point.Y = 41
 $LoginButton.Location = $System_Drawing_Point
 $LoginButton.Name = "LoginButton"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 23
 $System_Drawing_Size.Width = 75
 $LoginButton.Size = $System_Drawing_Size
 $LoginButton.TabIndex = 6
 $LoginButton.Text = "Login"
 $LoginButton.UseVisualStyleBackColor = $True
 $LoginButton.add_Click($LoginButton_OnClick)
 
 $groupBox1.Controls.Add($LoginButton)
 
 $Password.DataBindings.DefaultDataSourceUpdateMode = 0
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 113
 $System_Drawing_Point.Y = 71
 $Password.Location = $System_Drawing_Point
 $Password.Name = "Password"
 $Password.PasswordChar = '*'
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 20
 $System_Drawing_Size.Width = 122
 $Password.Size = $System_Drawing_Size
 $Password.TabIndex = 5
 
 $groupBox1.Controls.Add($Password)
 
 $User.DataBindings.DefaultDataSourceUpdateMode = 0
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 113
 $System_Drawing_Point.Y = 44
 $User.Location = $System_Drawing_Point
 $User.Name = "User"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 20
 $System_Drawing_Size.Width = 122
 $User.Size = $System_Drawing_Size
 $User.TabIndex = 4
 
 $groupBox1.Controls.Add($User)
 
 $vCenter.DataBindings.DefaultDataSourceUpdateMode = 0
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 113
 $System_Drawing_Point.Y = 17
 $vCenter.Location = $System_Drawing_Point
 $vCenter.Name = "vCenter"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 20
 $System_Drawing_Size.Width = 122
 $vCenter.Size = $System_Drawing_Size
 $vCenter.TabIndex = 3
 
 $groupBox1.Controls.Add($vCenter)
 
 $label3.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 7
 $System_Drawing_Point.Y = 74
 $label3.Location = $System_Drawing_Point
 $label3.Name = "label3"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 23
 $System_Drawing_Size.Width = 100
 $label3.Size = $System_Drawing_Size
 $label3.TabIndex = 2
 $label3.Text = "Password"
 
 $groupBox1.Controls.Add($label3)
 
 $label2.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 7
 $System_Drawing_Point.Y = 47
 $label2.Location = $System_Drawing_Point
 $label2.Name = "label2"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 23
 $System_Drawing_Size.Width = 100
 $label2.Size = $System_Drawing_Size
 $label2.TabIndex = 1
 $label2.Text = "User Name"
 
 $groupBox1.Controls.Add($label2)
 
 $label1.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 7
 $System_Drawing_Point.Y = 20
 $label1.Location = $System_Drawing_Point
 $label1.Name = "label1"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 23
 $System_Drawing_Size.Width = 100
 $label1.Size = $System_Drawing_Size
 $label1.TabIndex = 0
 $label1.Text = "VI Server"
 
 $groupBox1.Controls.Add($label1)
 
 
 
 $InitialFormWindowState = $form1.WindowState
 $form1.add_Load($OnLoadForm_StateCorrection)
 $form1.ShowDialog()| Out-Null
 
 
 
 
 
 GenerateForm
`

