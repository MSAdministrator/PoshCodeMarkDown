---
Author: andre combrinck
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5591
Published Date: 2015-11-16t07
Archived Date: 2015-01-31t20
---

# certificator part 2 - 

## Description

generate your own ca and sign your own certificates

## Comments



## Usage



## TODO



## function

`form-home`

## Code

`#
 #
 
         If ($Source -eq "btnCustomCert")
             {
             #---------------
             $btnPFXFileBrowse = New-Object System.Windows.Forms.Button
             $Global:tbPFXFile = New-Object System.Windows.Forms.TextBox 
             $dlgPFXFile = New-Object System.Windows.Forms.OpenFileDialog
 
                 $btnPFXFileBrowse.Size = New-Object System.Drawing.Size($lblWidth,$lblHeight)
                 $btnPFXFileBrowse.Top = $btnSigningCAkeyBrowse.Bottom + $gap
                 $btnPFXFileBrowse.Left = $lblIntroduction.Left
                 $btnPFXFileBrowse.Text = "Path to .pfx file"
 
                 $Global:tbPFXFile.Size = New-Object System.Drawing.Size($tbWidth,$tbHeight)
                 $Global:tbPFXFile.Top = $btnPFXFileBrowse.Top + 5
                 $Global:tbPFXFile.Left = $btnPFXFileBrowse.Right + 10
                 $Global:tbPFXFile.Text = ""
 
                 $dlgPFXFile.Title = "Specify the .pfx file that you would like to import:" ;
                 $dlgPFXFile.InitialDirectory = $Global:hshDefaultValues.Item("RootFolderPath")
                 $dlgPFXFile.Filter = "Personal Information Exchange files (*.pfx)|*.pfx|All files (*.*)|*.*"
 
                 $btnPFXFileBrowse.Add_Click({
                     If ($dlgPFXFile.ShowDialog() -eq "OK")
                         {
                         $Global:tbPFXFile.Text = $dlgPFXFile.FileName
                         }
                     })
 
             $objForm.Controls.Add($btnPFXFileBrowse)
             $objForm.Controls.Add($Global:tbPFXFile) 
             }
 
         #---------------
         $lblKeyPass = New-Object System.Windows.Forms.Label
         $tbKeyPass = New-Object System.Windows.Forms.TextBox
 
             $lblKeyPass.Size = New-Object System.Drawing.Size($lblWidth,$lblHeight)
             If ($Source -eq "btnNewCA")
                 {
                 $lblKeyPass.Top = $btnSigningCAcrtBrowse.Bottom + $gap
                 }
             ElseIf ($Source -eq "btnCustomCert")
                 {
                 $lblKeyPass.Top = $btnPFXFileBrowse.Bottom + $gap
                 }
             Else
                 {
                 $lblKeyPass.Top = $btnSigningCAkeyBrowse.Bottom + $gap
                 }
             $lblKeyPass.Left = $lblIntroduction.Left
             If ($Source -eq "btnVCSACerts" -or $Source -eq "btnDefaults" -or $Source -eq "btnESXiCerts")
             {
             $lblKeyPass.Text = "root Password"
             }
             Else
             {
             $lblKeyPass.Text = "Passphrase"
             }
 
             $tbKeyPass.Size = New-Object System.Drawing.Size($tbWidth,$tbHeight)
             $tbKeyPass.Top = $lblKeyPass.Top
             $tbKeyPass.Left = $lblKeyPass.Right + 10
             If ($Source -eq "btnVCSACerts" -or $Source -eq "btnDefaults" -or $Source -eq "btnESXiCerts")
                 {
                 $tbKeyPass.Text = $Global:hshDefaultValues.Item("RootPassword")
                 }
             Else
                 {
                 $tbKeyPass.Text = $Global:hshDefaultValues.Item("Passphrase")
                 }
             $tbKeyPass.PasswordChar = "*"
 
         $objForm.Controls.Add($lblKeyPass) 
         $objForm.Controls.Add($tbKeyPass) 
 
         #---------------
         If ($Source -eq "btnVCSACerts" -or $Source -eq "btnDefaults")
             {
             $lblVCSAPass = New-Object System.Windows.Forms.Label
             $tbVCSAPass = New-Object System.Windows.Forms.TextBox
 
                 $lblVCSAPass.Size = New-Object System.Drawing.Size($lblWidth,$lblHeight)
                 $lblVCSAPass.Top = $lblKeyPass.Bottom + $gap
                 $lblVCSAPass.Left = $lblIntroduction.Left
                 $lblVCSAPass.Text = "SSO Administrator Password"
 
                 $tbVCSAPass.Size = New-Object System.Drawing.Size($tbWidth,$tbHeight)
                 $tbVCSAPass.Top = $lblVCSAPass.Top
                 $tbVCSAPass.Left = $lblVCSAPass.Right + 10
                 $tbVCSAPass.Text = $Global:hshDefaultValues.Item("SSOPassword")
                 $tbVCSAPass.PasswordChar = "*"
 
             $objForm.Controls.Add($lblVCSAPass) 
             $objForm.Controls.Add($tbVCSAPass) 
             }
 
         #---------------
         $btnGenerate = New-Object System.Windows.Forms.Button
 
             $btnGenerate.Width = ($lblWidth + $tbWidth)/3
             $btnGenerate.Height = $lblHeight
             If ($Source -eq "btnVCSACerts" -or $Source -eq "btnDefaults")
             {
             $btnGenerate.Top = $lblVCSAPass.Bottom + $gap +20
             }
             Else
             {
             $btnGenerate.Top = $lblKeyPass.Bottom + $gap + 20
             }
             $btnGenerate.Left = $lblIntroduction.Left
             If ($Source -eq "btnDefaults")
                 {
                 $btnGenerate.Text = "Save to this Session"
                 }
             Else
                 {
                 $btnGenerate.Text = "Generate"
                 }
 
             $btnGenerate.Add_Click({
                 If ($Source -eq "btnNewCA")
                     {
                     Create-CAwithKey `
                         -OrganizationName $tbCompanyName_ws.Text`
                         -RootFolderPath $tbRootFolder.Text`
                         -OrganizationalUnitName $tbCompanyName_ws.Text`
                         -Hostname $tbhostname.Text`
                         -FQDN $tbFQDN.Text`
                         -CountryCode $tbCountryCode.Text`
                         -StateOrProvinceName $tbStateOrProvinceName.Text`
                         -LocalityCity $tbLocalityCity.Text`
                         -Passphrase $tbKeyPass.Text`
                         -NumberOfBits ($tbNumberOfBits.Text -as [int]) `
                         -NumOfDays 3650
                     }
                 ElseIf ($Source -eq "btnVCSACerts")
                     {
                     New-VCSA55Certificates `
                         -organizationName $tbCompanyName_ws.Text`
                         -RootFolderPath $tbRootFolder.Text`
                         -hostname $tbhostname.Text`
                         -IPaddress $tbIPaddress.Text`
                         -FQDN $tbFQDN.Text`
                         -CountryCode $tbCountryCode.Text`
                         -StateOrProvinceName $tbStateOrProvinceName.Text`
                         -LocalityCity $tbLocalityCity.Text`
                         -CAStorePath $tbSigningCAdir.Text `
                         -PathToCAKeyFile $tbSigningCAkey.Text`
                         -PathToCACRTFile $tbSigningCAcrt.Text`
                         -Password $tbKeyPass.Text
                     }
                 ElseIf ($Source -eq "btnESXiCerts")
                     {
                     New-ESXi55Certificates `
                         -OrganizationName $tbCompanyName_ws.Text`
                         -RootFolderPath $tbRootFolder.Text`
                         -FQDN $tbFQDN.Text`
                         -CountryCode $tbCountryCode.Text`
                         -StateOrProvinceName $tbStateOrProvinceName.Text`
                         -LocalityCity $tbLocalityCity.Text`
                         -CAStorePath $tbSigningCAdir.Text `
                         -PathToCAKeyFile $tbSigningCAkey.Text`
                         -PathToCACRTFile $tbSigningCAcrt.Text
                     }
                 ElseIf ($Source -eq "btnCustomCert")
                     {
                     New-ServerCertificate `
                         -OrganizationName $tbCompanyName_ws.Text`
                         -RootFolderPath $tbRootFolder.Text`
                         -FQDN $tbFQDN.Text`
                         -CountryCode $tbCountryCode.Text`
                         -StateOrProvinceName $tbStateOrProvinceName.Text`
                         -LocalityCity $tbLocalityCity.Text`
                         -CAStorePath $tbSigningCAdir.Text `
                         -PathToCAKeyFile $tbSigningCAkey.Text`
                         -PathToCACRTFile $tbSigningCAcrt.Text`
                         -ServerName $tbhostname.Text`
                         -ServerIP $tbIPaddress.Text`
                         -Passphrase $tbKeyPass.Text`
                         -NumberOfBits ($tbNumberOfBits.Text -as [int])
                     }
                 ElseIf ($Source -eq "btnDefaults")
                     {
                     $Global:hshDefaultValues["RootFolderPath"] = $tbRootFolder.Text
                     $Global:hshDefaultValues["Company"] = $tbCompanyName_ws.Text
                     $Global:hshDefaultValues["FQDN"] = $tbFQDN.Text
                     $Global:hshDefaultValues["Countrycode"] = $tbCountryCode.Text
                     $Global:hshDefaultValues["StateOrProvinceName"] = $tbStateOrProvinceName.Text
                     $Global:hshDefaultValues["LocalityCity"] = $tbLocalityCity.Text
                     $Global:hshDefaultValues["NumberOfBits"] = $tbNumberOfBits.Text
                     $Global:hshDefaultValues["vCenterHostname"] = $tbhostname.Text
                     $Global:hshDefaultValues["vCenterIPAddress"] = $tbIPaddress.Text
                     $Global:hshDefaultValues["SSOPassword"] = $tbVCSAPass.Text
                     $Global:hshDefaultValues["RootPassword"] = $tbKeyPass.Text
                     $Global:hshDefaultValues["SigningCARoot"] = $tbSigningCAdir.Text
                     $Global:hshDefaultValues["SigningCACRT"] = $tbSigningCAcrt.Text
                     $Global:hshDefaultValues["SigningCAKey"] = $tbSigningCAkey.Text
                     $objForm.Close()
                     }
             })
 
         $objForm.Controls.Add($btnGenerate)
 
         #---------------
         $btnCloseForm = New-Object System.Windows.Forms.Button
 
             $btnCloseForm.Top = $btnGenerate.Top
             $btnCloseForm.Left = $btnGenerate.Right + 5
             $btnCloseForm.Width = $btnGenerate.Width
             $btnCloseForm.Height = $btnGenerate.Height
             $btnCloseForm.Text = "Close"
 
             $btnCloseForm.Add_Click({
                 $objForm.Close()
                 })
 
         $objForm.Controls.Add($btnCloseForm)
 
         #---------------
         $btnUpload = New-Object System.Windows.Forms.Button
 
             $btnUpload.Top = $btnGenerate.Top
             $btnUpload.Left = $btnCloseForm.Right + 5
             $btnUpload.Width = $btnGenerate.Width
             $btnUpload.Height = $btnGenerate.Height
             If ($Source -eq "btnNewCA")
                 {
                 $btnUpload.Text = "Trust this Root CA"
                 }
             ElseIf ($Source -eq "btnDefaults")
                 {
                 $btnUpload.Text = "Save to Profile"
                 }
             ElseIf ($Source -eq "btnVCSACerts")
                 {
                 $btnUpload.Text = "Upload to vCenter"
                 }
             ElseIf ($Source -eq "btnESXiCerts")
                 {
                 $btnUpload.Text = "Upload to ESXi Hosts`r`n(Work in Progress)"
                 }
             ElseIf ($Source -eq "btnCustomCert")
                 {
                 $btnUpload.Text = "Import the .pfx"
                 }
 
             $btnUpload.Add_Click({
                 If ($Global:strWinSCPExecutablePath -eq "" -and ($Source -eq "btnVCSACerts" -or $Source -eq "btnESXiCerts"))
                     {
                     $urlInstallWinSCP = "http://winscp.net/eng/download.php"
                     Start-Process $urlInstallWinSCP
                     $objForm.Close()
                     }
                 ElseIf ($Source -eq "btnNewCA")
                     {
                     Write-Host "Installing the "$tbCompanyName_ws.Text" Root CA into the computer's Trusted Root Certification Authorities list."
                     Import-Certificate -FilePath $tbSigningCAcrt.Text -CertStoreLocation Cert:\LocalMachine\Root
                     }
                 ElseIf ($Source -eq "btnVCSACerts")
                     {
                     Upload-vCenterCertificates -SSOAdminPassword $tbVCSAPass.Text -LookupServiceIP $tbIPaddress.Text -rootPassword $tbKeyPass.Text -RootFolderPath $tbRootFolder.Text
                     }
                 ElseIf ($Source -eq "btnCustomCert")
                     {
                     Import-PfxCertificate -FilePath $tbPFXFile.Text -CertStoreLocation Cert:\LocalMachine\My -Password (ConvertTo-SecureString $tbKeyPass.Text -AsPlainText -Force) -Exportable
                     }
                 ElseIf ($Source -eq "btnDefaults")
                     {
                     $strXMLFileName = $tbCompanyName_ws.Text + '.xml'
                     $strPathToCertificatorFolder = "$env:APPDATA\Certificator"
                     If (-Not (Test-Path -Path $strPathToCertificatorFolder))
                         {
                         New-Item -Path $strPathToCertificatorFolder -ItemType Directory
                         }
                     $Global:hshDefaultValues["RootFolderPath"] = $tbRootFolder.Text
                     $Global:hshDefaultValues["Company"] = $tbCompanyName_ws.Text
                     $Global:hshDefaultValues["FQDN"] = $tbFQDN.Text
                     $Global:hshDefaultValues["Countrycode"] = $tbCountryCode.Text
                     $Global:hshDefaultValues["StateOrProvinceName"] = $tbStateOrProvinceName.Text
                     $Global:hshDefaultValues["LocalityCity"] = $tbLocalityCity.Text
                     $Global:hshDefaultValues["NumberOfBits"] = $tbNumberOfBits.Text
                     $Global:hshDefaultValues["vCenterHostname"] = $tbhostname.Text
                     $Global:hshDefaultValues["vCenterIPAddress"] = $tbIPaddress.Text
                     $Global:hshDefaultValues["SSOPassword"] = $tbVCSAPass.Text
                     $Global:hshDefaultValues["RootPassword"] = $tbKeyPass.Text
                     $Global:hshDefaultValues["SigningCARoot"] = $tbSigningCAdir.Text
                     $Global:hshDefaultValues["SigningCACRT"] = $tbSigningCAcrt.Text
                     $Global:hshDefaultValues["SigningCAKey"] = $tbSigningCAkey.Text
                     $Global:hshDefaultValues
                     $Global:hshDefaultValues | Export-Clixml "$strPathToCertificatorFolder\$strXMLFileName"
                     Write-Host "Default values were saved to $strPathToCertificatorFolder\$strXMLFileName"
                     }
                })
 
         $objForm.Controls.Add($btnUpload)
 
         #---------------
         $lblCredit = New-Object System.Windows.Forms.Label
 
             $lblCredit.Top = $btnGenerate.Bottom + 2
             $lblCredit.Left = $lblIntroduction.Left
             $lblCredit.Width = 150
             $lblCredit.Height = 30 
             $lblCredit.Text = "Created by Andre Combrinck" 
 
         $objForm.Controls.Add($lblCredit)
 
     $objForm.KeyPreview = $True
     $objForm.Add_KeyDown({
         If ($_.KeyCode -eq "Escape")
             {
                 $objForm.Close()
             }
         })
     $objForm.Add_Shown({$objForm.Activate()})
     [void] $objForm.ShowDialog()
 }
 
 #---------------
 #---------------
 Function Form-Home
 {
     [void] [System.Reflection.Assembly]::LoadWithPartialName("System.Drawing")
     [void] [System.Reflection.Assembly]::LoadWithPartialName("System.Windows.Forms") 
 
     #---------------
     $objHomeForm = New-Object System.Windows.Forms.Form 
 
         $objHomeForm.KeyPreview = $True
 
         [int]$gap = 4
         $objHomeForm.Text = "Certificator"
         $objHomeForm.AutoSize = $True
         $objHomeForm.StartPosition = "CenterScreen"
 
 
         #---------------
         $lblIntroduction = New-Object System.Windows.Forms.Label
 
             $lblIntroduction.Location = New-Object System.Drawing.Size(10,10)
             $lblIntroduction.Width = $objHomeForm.Width
             $lblIntroduction.Height = 70 
             $lblIntroduction.Text = "This script is provided as is.`r`nFor the most part, it is safe to run.`r`nBefore using any of the buttons at the bottom right on a production system, make sure you have tested it first and understand the impact it will have on your environment." 
 
         $objHomeForm.Controls.Add($lblIntroduction)
 
         #---------------
         $lblOpenSSLCheck = New-Object System.Windows.Forms.Label
 
             $lblOpenSSLCheck.Top = $lblIntroduction.Bottom + $gap
             $lblOpenSSLCheck.Left = $lblIntroduction.Left
             $lblOpenSSLCheck.Width = $objHomeForm.Width - 20
             $lblOpenSSLCheck.Height = 40 
 
             $arrOpenSSLCheckResult = @(Check-OpenSSL)
             If (-Not $arrOpenSSLCheckResult[0])
                 {
                 $lblOpenSSLCheck.ForeColor = "Red"
                 $lblOpenSSLCheck.Text = $arrOpenSSLCheckResult[1]
                 }
             Else
                 {
                 $lblOpenSSLCheck.ForeColor = "Green"
                 $lblOpenSSLCheck.Text = $arrOpenSSLCheckResult[1]
                 }
 
         $objHomeForm.Controls.Add($lblOpenSSLCheck) 
 
         #---------------
         $lblWinSCPCheck = New-Object System.Windows.Forms.Label
 
             $lblWinSCPCheck.Top = $lblOpenSSLCheck.Bottom + $gap
             $lblWinSCPCheck.Left = $lblIntroduction.Left
             $lblWinSCPCheck.Width = $objHomeForm.Width - 20
             $lblWinSCPCheck.Height = 40 
 
             $arrWinSCPCheckResult = @(Check-WinSCP)
             $lblWinSCPCheck.ForeColor = $arrWinSCPCheckResult[2]
             $lblWinSCPCheck.Text = $arrWinSCPCheckResult[1]
 
         $objHomeForm.Controls.Add($lblWinSCPCheck) 
 
         #---------------
         $btnDefaults = New-Object System.Windows.Forms.Button
 
             $btnDefaults.Top = $lblWinSCPCheck.Bottom + $gap
             $btnDefaults.Left = $lblIntroduction.Left
             $btnDefaults.Width = 200
             $btnDefaults.Height = 40
             $btnDefaults.Text = "Specify Default Values"
 
             $btnDefaults.Add_Click({
                 If ((Get-ChildItem $env:APPDATA\Certificator -ErrorAction SilentlyContinue).Count -ge 1)
                     {
                     Write-Host "Looking up previously saved default files"
                     Get-CertificatorDefaults
                     }
                 Form-CertificateInfo -Source "btnDefaults"
                 })
 
         $objHomeForm.Controls.Add($btnDefaults)
 
         #---------------
         $btnNewCA = New-Object System.Windows.Forms.Button
 
             $btnNewCA.Top = $btnDefaults.Bottom + $gap
             $btnNewCA.Left = $lblIntroduction.Left
             $btnNewCA.Width = 200
             $btnNewCA.Height = 40
             $btnNewCA.Text = "Generate a Certificate Authority"
 
             $btnNewCA.Add_Click({
                 Form-CertificateInfo -Source "btnNewCA"
                 })
 
         $objHomeForm.Controls.Add($btnNewCA)
 
         #---------------
         $btnVCSACerts = New-Object System.Windows.Forms.Button
 
             $btnVCSACerts.Top = $btnNewCA.Bottom + 10
             $btnVCSACerts.Left = $lblIntroduction.Left
             $btnVCSACerts.Width = 200
             $btnVCSACerts.Height = 40
             $btnVCSACerts.Text = "Generate vCenter Server Certificates"
 
             $btnVCSACerts.Add_Click({
                 Form-CertificateInfo -Source "btnVCSACerts"
                 })
 
         $objHomeForm.Controls.Add($btnVCSACerts)
 
         #---------------
         $btnESXiCerts = New-Object System.Windows.Forms.Button
 
             $btnESXiCerts.Top = $btnVCSACerts.Bottom + 10
             $btnESXiCerts.Left = $lblIntroduction.Left
             $btnESXiCerts.Width = 200
             $btnESXiCerts.Height = 40
             $btnESXiCerts.Text = "Generate ESXi Hosts Certificates"
 
             $btnESXiCerts.Add_Click({
                 Form-CertificateInfo -Source "btnESXiCerts"
                 })
 
         $objHomeForm.Controls.Add($btnESXiCerts)
 
         #---------------
         $btnCustomCert = New-Object System.Windows.Forms.Button
 
             $btnCustomCert.Top = $btnESXiCerts.Bottom + 10
             $btnCustomCert.Left = $lblIntroduction.Left
             $btnCustomCert.Width = 200
             $btnCustomCert.Height = 40
             $btnCustomCert.Text = "Generate a custom certificate"
 
             $btnCustomCert.Add_Click({
                 Form-CertificateInfo -Source "btnCustomCert"
                 })
 
         $objHomeForm.Controls.Add($btnCustomCert)
 
 
         #---------------
         $btnClose = New-Object System.Windows.Forms.Button
 
             $btnClose.Bounds = New-Object System.Drawing.Rectangle(10,500,200,40)
             $btnClose.Top = $btnCustomCert.Bottom + 20
             $btnClose.Left = $lblIntroduction.Left
             $btnClose.Width = 200
             $btnClose.Height = 40
             $btnClose.Text = "Close"
 
             $btnClose.Add_Click(
                 {
                 $objHomeForm.Close()
                 })
 
         $objHomeForm.Controls.Add($btnClose)
 
         #---------------
         $lblCredit = New-Object System.Windows.Forms.Label
 
             $lblCredit.Top = $btnClose.Bottom + 2
             $lblCredit.Left = $lblIntroduction.Left
             $lblCredit.Width = 150
             $lblCredit.Height = 30 
             $lblCredit.Text = "Created by Andre Combrinck" 
 
         $objHomeForm.Controls.Add($lblCredit)
     
     $objHomeForm.Add_KeyDown({
         If ($_.KeyCode -eq "Escape")
             {
             $objHomeForm.Close()
             }
         })
 
     $objHomeForm.Add_Shown({$objHomeForm.Activate()})
     [void] $objHomeForm.ShowDialog()
 }
 If (-Not $Script)
     {
     Form-Home
     }
`

