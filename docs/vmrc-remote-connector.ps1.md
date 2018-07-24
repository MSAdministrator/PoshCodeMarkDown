---
Author: patrick
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 1527
Published Date: 2010-12-13t23
Archived Date: 2016-05-20t09
---

# vmrc remote connector - 

## Description

connects with vmrc to an esx farm. now with option to allow multiple connections to one vm

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $script:viserver = "esx.local"
 $script:vmrc = "C:\Program Files\VMwareVMRC\vmware-vmrc.exe"
 $script:vm = ""
 $script:vmt = ""
 $script:vmcon = ""
 $script:selectvm = ""
 
 $vmcon = Connect-VIServer -Server $script:viserver -Protocol https -Credential (Get-Credential)
 if (!$vmcon) {
 	Write-Output "No Connection!"
 	break
 }
 
 $script:vmt = Get-VM 
 
 function GenerateForm {
 
 [reflection.assembly]::loadwithpartialname("System.Windows.Forms") | Out-Null
 [reflection.assembly]::loadwithpartialname("System.Drawing") | Out-Null
 
 $form1 = New-Object System.Windows.Forms.Form
 $btnConnect = New-Object System.Windows.Forms.Button
 $txtEntity = New-Object System.Windows.Forms.TextBox
 $chkIP = New-Object System.Windows.Forms.CheckBox
 $comboBox1 = New-Object System.Windows.Forms.ComboBox
 
 $btnConnect_OnClick= 
 {
 
 if (!$txtEntity.Text -eq "" -and !$comboBox1.Text -eq "") 
 {
 	Write-Host "Bitte nur eine Option anwählen!"
 	break
 } 
 
 if ($txtEntity.Text -eq "") {
 	$script:selectvm = $comboBox1.Text
 } else { 
 	$script:selectvm = $txtEntity.Text
 }
 
 if ($chkIP.Checked) { 
 		$script:chk = "FALSE"
 
 		foreach ($script:vm in $script:vmt) {
 			$vmg = Get-VMGuest $script:vm
 			
 			if ($vmg.IPAddress -eq $script:selectvm) {
 				$script:chk = ""
 				break
 			} 
 		}
 	}
 	
 If (!$script:selectvm -eq "" -and !$chkIP.Checked)  {
 		$script:vm = Get-VM $script:selectvm
 	
 		if($script:vm.Count) {
 			Write-Output "Define only one host!"
 			break
 		} else {
 			if (!$script:vm.id) { 
 				Write-Output "No VM found"
 				$script:chk = "FALSE";
 				break
 			}
 
 		}
 	}
 	
 	if (!$script:chk) {
 		if (!$script:vm -eq "") {
 			if (!($script:gvm = Get-View $script:vm.id) -eq "") {
 					if (($multiple) -or (!($multiple) -and $script:gvm.Runtime.NumMksConnections -lt 1)) {
 						$vmrcargs = '-h ' + $script:vm.Host + ' -m "' + $script:gvm.Config.Files.VmPathName + '"'
 						[Diagnostics.Process]::Start($script:vmrc, $vmrcargs)
 					}
  			}
 		}
 	}
 }
 
 #----------------------------------------------
 $form1.Text = 'VMware Remote Connector'
 $form1.Name = 'form1'
 $form1.DataBindings.DefaultDataSourceUpdateMode = 0
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 193
 $System_Drawing_Size.Width = 281
 $form1.ClientSize = $System_Drawing_Size
 $form1.FormBorderStyle = 1
 
 $btnConnect.TabIndex = 2
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 23
 $System_Drawing_Size.Width = 75
 $btnConnect.Size = $System_Drawing_Size
 $btnConnect.Name = 'btnConnect'
 $btnConnect.UseVisualStyleBackColor = $True
 
 $btnConnect.Text = 'Verbinden'
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 97
 $System_Drawing_Point.Y = 115
 $btnConnect.Location = $System_Drawing_Point
 
 $btnConnect.DataBindings.DefaultDataSourceUpdateMode = 0
 $btnConnect.add_Click($btnConnect_OnClick)
 
 $form1.Controls.Add($btnConnect)
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 12
 $System_Drawing_Point.Y = 17
 $txtEntity.Location = $System_Drawing_Point
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 20
 $System_Drawing_Size.Width = 261
 $txtEntity.Size = $System_Drawing_Size
 $txtEntity.DataBindings.DefaultDataSourceUpdateMode = 0
 $txtEntity.Name = 'txtEntity'
 $txtEntity.TabIndex = 1
 
 $form1.Controls.Add($txtEntity)
 
 $chkIP.TabIndex = 0
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 19
 $System_Drawing_Size.Width = 160
 $chkIP.Size = $System_Drawing_Size
 $chkIP.Name = 'chkIP'
 $chkIP.UseVisualStyleBackColor = $True
 
 $chkIP.Text = 'IP Adresse'
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 12
 $System_Drawing_Point.Y = 43
 $chkIP.Location = $System_Drawing_Point
 
 $chkIP.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $form1.Controls.Add($chkIP)
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 12
 $System_Drawing_Point.Y = 73
 $comboBox1.Location = $System_Drawing_Point
 $comboBox1.DataBindings.DefaultDataSourceUpdateMode = 0
 $comboBox1.FormattingEnabled = $True
 $comboBox1.Name = 'comboBox1'
 $comboBox1.TabIndex = 0
 foreach ($cvm in $script:vmt) {
 	$comboBox1.Items.Add($cvm.Name) | Out-Null
 }
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 20
 $System_Drawing_Size.Width = 261
 $comboBox1.Size = $System_Drawing_Size
 
 $form1.Controls.Add($comboBox1)
 
 
 $form1.ShowDialog()| Out-Null
 
 }
 
 GenerateForm
`

