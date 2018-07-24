---
Author: adrian woodrup
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 4071
Published Date: 2014-04-04t17
Archived Date: 2017-03-16t01
---

# office 365 - get email - 

## Description

i wrote this for our first line team, they aren’t the best with powershell so i wrote a gui that would help them find which user has a specific email address in office 365.

## Comments



## Usage



## TODO



## script

`show-powershell`

## Code

`#
 #
 $Content = '$script:showWindowAsync = Add-Type �memberDefinition @�','[DllImport("user32.dll")]','public static extern bool ShowWindowAsync(IntPtr hWnd, int nCmdShow);','�@ -name �Win32ShowWindowAsync� -namespace Win32Functions �passThru','function Show-PowerShell() {','$null = $showWindowAsync::ShowWindowAsync((Get-Process �id $pid).MainWindowHandle, 10)','}','function Hide-PowerShell() {','$null = $showWindowAsync::ShowWindowAsync((Get-Process �id $pid).MainWindowHandle, 2)','}'
 Set-Content $Content -Path C:\
 
 Add-Module PowerShell
 Hide-PowerShell 
 
 Import-Module ActiveDirectory 
 Import-Module MSOnline
 
 $cred = Get-Credential 
 Connect-MsolService -Credential $cred
 $session = New-PSSession -ConfigurationName Microsoft.Exchange -ConnectionUri https://ps.outlook.com/powershell/ -Credential $Cred -Authentication Basic -AllowRedirection
 Import-PSSession $session
 
 [void] [System.Reflection.Assembly]::LoadWithPartialName("System.Drawing") 
 [void] [System.Reflection.Assembly]::LoadWithPartialName("System.Windows.Forms") 
 
 $objForm = New-Object System.Windows.Forms.Form 
 $objForm.Text = "Check Email address in Office365"
 $objForm.Size = New-Object System.Drawing.Size(300,200) 
 $objForm.StartPosition = "CenterScreen"
 
 $objForm.KeyPreview = $True
 $objForm.Add_KeyDown({if ($_.KeyCode -eq "Enter") 
     {$x=$objTextBox.Text;$objForm.Close()}})
 $objForm.Add_KeyDown({if ($_.KeyCode -eq "Escape") 
     {$objForm.Close()}})
 
 $OKButton = New-Object System.Windows.Forms.Button
 $OKButton.Location = New-Object System.Drawing.Size(75,120)
 $OKButton.Size = New-Object System.Drawing.Size(75,23)
 $OKButton.Text = "OK"
 $OKButton.Add_Click({$x=$objTextBox.Text;$objForm.Close()})
 $objForm.Controls.Add($OKButton)
 
 $CancelButton = New-Object System.Windows.Forms.Button
 $CancelButton.Location = New-Object System.Drawing.Size(150,120)
 $CancelButton.Size = New-Object System.Drawing.Size(75,23)
 $CancelButton.Text = "Cancel"
 $CancelButton.Add_Click({$objForm.Close()})
 $objForm.Controls.Add($CancelButton)
 
 $objLabel = New-Object System.Windows.Forms.Label
 $objLabel.Location = New-Object System.Drawing.Size(10,20) 
 $objLabel.Size = New-Object System.Drawing.Size(280,20) 
 $objLabel.Text = "Please enter the email address below:"
 $objForm.Controls.Add($objLabel) 
 
 $objTextBox = New-Object System.Windows.Forms.TextBox 
 $objTextBox.Location = New-Object System.Drawing.Size(10,40) 
 $objTextBox.Size = New-Object System.Drawing.Size(260,20) 
 $objForm.Controls.Add($objTextBox) 
 
 $objForm.Topmost = $True
 
 $objForm.Add_Shown({$objForm.Activate()})
 [void] $objForm.ShowDialog()
 
 $x
 $EmailAddress = 'smtp:' + $x
 $Answer = Get-Mailbox | Where-Object {$_.EmailAddresses -eq $EmailAddress} | Select-Object 'Name'
 
 $objForm2 = New-Object System.Windows.Forms.Form 
 $objForm2.Text = "The Email address is asigned to account:"
 $objForm2.Size = New-Object System.Drawing.Size(300,200) 
 $objForm2.StartPosition = "CenterScreen"
 
 $objLabel2 = New-Object System.Windows.Forms.Label
 $objLabel2.Location = New-Object System.Drawing.Size(10,20) 
 $objLabel2.Size = New-Object System.Drawing.Size(280,20) 
 $objLabel2.Text = $Answer
 $objForm2.Controls.Add($objLabel2) 
 
 $OKButton2 = New-Object System.Windows.Forms.Button
 $OKButton2.Location = New-Object System.Drawing.Size(75,120)
 $OKButton2.Size = New-Object System.Drawing.Size(75,23)
 $OKButton2.Text = "OK"
 $OKButton2.Add_Click({$objForm2.Close()})
 $objForm2.Controls.Add($OKButton2)
 
 $objForm2.Add_Shown({$objForm2.Activate()})
 [void] $objForm2.ShowDialog()
 
 $sessionName = Get-PSSession | Select-Object 'Name'
 Remove-PSSession -Name $sessionName
`

