---
Author: andre combrinck
Publisher: 
Copyright: 
Email: 
Version: 5.2
Encoding: ascii
License: cc0
PoshCode ID: 5593
Published Date: 2015-11-16t07
Archived Date: 2015-05-05t00
---

# certificator part 1 - 

## Description

generate your own ca and sign your own certificates

## Comments



## Usage



## TODO



## script

`check-openssl`

## Code

`#
 #
 Param
     (
     [switch]$Script
     )
 
 #---------------
 #---------------
 [hashtable]$Global:hshDefaultValues = @{}
 $Global:hshDefaultValues = @{
     OpenSSLVersion = "0.9.8"
     WinSCPVersion = 5.2
     RootFolderPath = "C:\Customers\A Test Company"
     Company = "My Company"
     FQDN = "company.co.za"
     Countrycode = "ZA"
     StateOrProvinceName = "Gauteng"
     LocalityCity = "Pretoria"
     NumberOfBits = 2048
     DaysToExpiry = 3650
     Passphrase = "qwer1234"
     vCenterHostname = "vCenter55"
     vCenterIPAddress = "10.0.0.10"
     SSOPassword = "P@ssw0rd"
     RootPassword = "vmware"
     }
 $Global:hshDefaultValues.Add("CertStorePath",$Global:hshDefaultValues.Item("RootFolderPath")+"\ssl")
 $Global:hshDefaultValues.Add("SigningCARoot",$Global:hshDefaultValues.Item("RootFolderPath")+"\ssl\ca")
 $Global:hshDefaultValues.Add("SigningCACRT",$Global:hshDefaultValues.Item("RootFolderPath")+"\ssl\ca\ca.crt")
 $Global:hshDefaultValues.Add("SigningCAKey",$Global:hshDefaultValues.Item("RootFolderPath")+"\ssl\ca\ca.key")
 
 #---------------
 #---------------
 Function Check-OpenSSL
 {
     Param 
         (
         [string]$OpenSSLVersion = $Global:hshDefaultValues.Item("OpenSSLVersion")
         )
 
     $Global:strWorkingDirectoryPath = ""
     $Global:strOpenSSLExecutablePath = ""
     $urlInstallOpenSSL = "http://kb.vmware.com/selfservice/search.do?cmd=displayKC&docType=kc&docTypeID=DT_KB_1_1&externalId=2015387"
 
     Write-Host "Checking if you have OpenSSL 32-bit Installed..."
     $strOpenSSLRegKey = "HKLM:Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\OpenSSL (32-bit)_is1\"
     $bolOpenSSLRegExist = Test-Path -Path $strOpenSSLRegKey
     If ($bolOpenSSLRegExist)
         {
         $strOpenSSLInstallPath = Get-ItemProperty -Path $strOpenSSLRegKey -Name "Inno Setup: App Path"
         $strInstallLocation = $strOpenSSLInstallPath."Inno Setup: App Path"
         $Global:strWorkingDirectoryPath = "$strInstallLocation\bin\"
         $Global:strOpenSSLExecutablePath = "$strInstallLocation\bin\openssl.exe"
         Write-Host "`tOpenSSL will run from: $Global:strOpenSSLExecutablePath"
         $strOpenSSLVersion = & $strInstallLocation\bin\openssl.exe version
 
         If ($strOpenSSLVersion.Contains($OpenSSLVersion))
             {
             Write-Host "`tOpenSSL is installed in $strInstallLocation and is the correct version of $strOpenSSLVersion" -ForegroundColor Green
             return @($strInstallLocation, "OpenSSL is installed in $strInstallLocation and is the correct version of $strOpenSSLVersion")
             }
         Else
             {
             Write-Host "`tOpenSSL is installed in $strInstallLocation but it needs to be version $OpenSSLVersion" -ForegroundColor Yellow
             Start-Process $urlInstallOpenSSL
             return $false, "OpenSSL is installed in $strInstallLocation but it needs to be version $OpenSSLVersion"
             }
         }
     Else
         {
         Write-Host "`tYou will need to install OpenSSL before you can continue" -ForegroundColor Red
         Start-Process $urlInstallOpenSSL
         return $false, "You will need to install OpenSSL before you can continue"
         }
 }
 
 #---------------
 #---------------
 Function Check-WinSCP
 {
     Param 
         (
         [single]$WinSCPVersion = $Global:hshDefaultValues.Item("WinSCPVersion")
         )
 
     Write-Host "Checking if you have WinSCP Installed..."
     $Global:strWinSCPExecutablePath = ""
     $strWinSCPRegKey = "HKLM:Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\winscp3_is1\"
     $bolWinSCPRegExist = Test-Path -Path $strWinSCPRegKey
     If ($bolWinSCPRegExist)
         {
         $strWinSCPInstallPath = Get-ItemProperty -Path $strWinSCPRegKey -Name "Inno Setup: App Path"
         $strInstallLocation = $strWinSCPInstallPath."Inno Setup: App Path"
         $Global:strWinSCPExecutablePath = "$strInstallLocation\WinSCP.exe"
         Write-Host "`tWinSCP will run from: $Global:strWinSCPExecutablePath"
 
         [single]$strWinSCPMajorVersion = (Get-ItemProperty -Path $strWinSCPRegKey -Name "MajorVersion").MajorVersion
         [single]$strWinSCPMinorVersion = (Get-ItemProperty -Path $strWinSCPRegKey -Name "MinorVersion").MinorVersion
         [single]$strWinSCPVersion = "$strWinSCPMajorVersion.$strWinSCPMinorVersion"
         $strColour = ""
         $strMessage = ""
 
         If ($strWinSCPVersion -ge $WinSCPVersion)
             {
             $strColour = "Green"
             $strMessage = "WinSCP is installed in $strInstallLocation and is the correct version of $strWinSCPVersion"
             Write-Host "   "$strMessage -ForegroundColor Green 
             }
         Else
             {
             $strColour = "Orange"
             $strMessage = "WinSCP is installed in $strInstallLocation, but should be version $WinSCPVersion or higher"
             Write-Host "   "$strMessage -ForegroundColor Orange
             }
         return @($strInstallLocation, $strMessage, $strColour)
        }
     Else
         {
         Write-Host "`tYou will need to install WinSCP before you can continue" -ForegroundColor Red
         return @($false, "If you need to upload the certificates after they are created, you will need to install WinSCP before you can continue", "Red")
         }
 }
 
 #---------------
 #---------------
 Function Test-PowerCLI
 	{
 	$objVVC = $null
 	$objVVC = Get-PSSnapin -Name VMware.VimAutomation.Core -ErrorAction SilentlyContinue
 	If ($objVVC -eq "VMware.VimAutomation.Core")
 		{
 	    $urlInstallPowerCLI = "https://my.vmware.com/web/vmware/details?downloadGroup=PCLI550&productId=352"
 	    Start-Process $urlInstallPowerCLI
         return $false
 		}
     Else
         {
 	    Add-PSSnapin VMware.VimAutomation.Core
         return $true
         }
 	}
 
 #---------------
 #---------------
 {
     Param (
         [string]$RootFolderPath = $Global:hshDefaultValues.Item("RootFolderPath")
         )
 
     [int]$count = 0
     Write-Host "Checking if the necessary folders exist for the certificates"
     If (Test-Path "$RootFolderPath\ssl")
         {
         If (Test-Path $RootFolderPath\ssl\ca){$count++}
         If (Test-Path $RootFolderPath\ssl\vpxd){$count++}
         If (Test-Path $RootFolderPath\ssl\inventoryservice){$count++}
         If (Test-Path $RootFolderPath\ssl\autodeploy){$count++}
         If (Test-Path $RootFolderPath\ssl\logbrowser){$count++}
         return $true,$count
         }
     Else
         {
         return $false,$count
         }
 }
 
 #---------------
 #---------------
 Function Create-CertStore
 {
     Param (
         [string]$RootFolderPath = $Global:hshDefaultValues.Item("RootFolderPath"),
         [switch]$CAonly,
         $Folder
         )
 
     [int]$count = 0
     If (-Not (Test-Path $RootFolderPath\ssl))
         {
         Write-Host "`tCreating $RootFolderPath\ssl"
         New-Item $RootFolderPath\ssl -ItemType Directory | Out-Null
         }
     If ($CAonly)
         {
         If (-Not (Test-Path $RootFolderPath\ssl\ca))
             {
             Write-Host "`tCreating $RootFolderPath\ssl\ca"
             Write-Host "`tCreating the necessary files for you CA"
             New-Item $RootFolderPath\ssl\ca -ItemType Directory | Out-Null
             New-Item $RootFolderPath\ssl\ca\certs -ItemType Directory | Out-Null
             New-Item $RootFolderPath\ssl\ca\newcerts -ItemType Directory | Out-Null
             New-Item $RootFolderPath\ssl\ca\crl -ItemType Directory | Out-Null
             New-Item $RootFolderPath\ssl\ca\.rnd -ItemType File | Out-Null
             New-Item $RootFolderPath\ssl\ca\index.txt -ItemType File | Out-Null
             New-Item $RootFolderPath\ssl\ca\index.txt.attr -ItemType File | Out-Null
             New-Item $RootFolderPath\ssl\ca\serial -ItemType File | Out-Null
             Set-Content -Path $RootFolderPath\ssl\ca\serial -Value "1000" | Out-Null
             New-Item $RootFolderPath\ssl\ca\crlnumber -ItemType File | Out-Null
             New-Item $RootFolderPath\ssl\ca\crl.pem -ItemType File | Out-Null
             }
         return "$RootFolderPath\ssl\$Folder"
         }
     Else
         {
         If (-Not (Test-Path "$RootFolderPath\ssl\$Folder"))
             {
             Write-Host "`tCreating $RootFolderPath\ssl\$Folder"
             New-Item "$RootFolderPath\ssl\$Folder" -ItemType Directory | Out-Null
             }
         return "$RootFolderPath\ssl\$Folder"
         }
 }
 
 #---------------
 #---------------
 Function Create-OpenSSLConfigFile
 {
     Param (
         [string]$ConfigFileName,
         [string]$KeyFileName,
         [int]$NumberOfBits = $Global:hshDefaultValues.Item("NumberOfBits"),
         [string]$KeyPass = $Global:hshDefaultValues.Item("Passphrase"),
         [string]$hostname = $Global:hshDefaultValues.Item("vCenterHostname"),
         [string]$IPaddress = $Global:hshDefaultValues.Item("vCenterIPAddress"),
         [string]$FQDN = $Global:hshDefaultValues.Item("FQDN"),
         [string]$commonName = "$hostname.$FQDN",
         [string]$CountryCode = $Global:hshDefaultValues.Item("Countrycode"),
         [string]$StateOrProvinceName = $Global:hshDefaultValues.Item("StateOrProvinceName"),
         [string]$LocalityCity = $Global:hshDefaultValues.Item("LocalityCity"),
         [string]$organizationName = $Global:hshDefaultValues.Item("Company"),
         [string]$organizationalUnitName,
         [string]$CAStorePath = $Global:hshDefaultValues.Item("RootFolderPath")+"\ssl\ca"
         )
 
     Write-Host "`tCreating the custom config file"
     $PathToKeyFile = "$CertStorePath\$KeyFileName"
     $subjectAltName = "IP:$IPaddress, DNS:$hostname.$FQDN, DNS:$hostname"
     $CAdir = ($CAStorePath).Replace("\","\\")
 
     $RequestFile = @"
 RANDFILE		= $CAdir\\.rnd
 
 [ req ]
 default_bits = $NumberOfBits
 default_keyfile = $PathToKeyFile
 distinguished_name = req_distinguished_name
 encrypt_key = no
 prompt = no
 string_mask = nombstr
 req_extensions = v3_req
 input_password = $KeyPass
 output_password = $KeyPass
 
 [ v3_req ]
 basicConstraints = CA:false
 keyUsage = digitalSignature, keyEncipherment, dataEncipherment
 extendedKeyUsage = serverAuth, clientAuth
 subjectAltName = $subjectAltName
 
 [ req_distinguished_name ]
 countryName = $CountryCode
 stateOrProvinceName = $StateOrProvinceName
 localityName = $LocalityCity
 0.organizationName = $organizationName
 organizationalUnitName = $organizationalUnitName
 commonName = $commonName
 
 [ ca ]
 
 [ CA_default ]
 copy_extensions = copy
 
 policy		= policy_match
 
 [ policy_match ]
 countryName		= match
 stateOrProvinceName	= optional
 organizationName	= match
 organizationalUnitName	= optional
 commonName		= supplied
 emailAddress		= optional
 
 
 [ req_attributes ]
 challengePassword		= A challenge password
 challengePassword_min		= 4
 challengePassword_max		= 20
 
 unstructuredName		= An optional company name
 
 [ usr_cert ]
 basicConstraints=CA:FALSE
 nsComment			= "Certificator by Andre Combrinck"
 subjectKeyIdentifier=hash
 authorityKeyIdentifier=keyid,issuer
 
 [ v3_ca ]
 subjectKeyIdentifier=hash
 authorityKeyIdentifier=keyid:always,issuer:always
 basicConstraints = CA:true
 "@
     Set-Content -Path "$CertStorePath\$ConfigFileName" -Value $RequestFile 
 }
 
 #---------------
 #---------------
 Function Test-PKCS1Key
 {
     Param (
         [string]$PathToKeyFile
         )
 
     Write-Host "`tChecking if the key is in PKCS1 format"
     If ((Get-Content $PathToKeyFile | select -First 1).Contains('BEGIN RSA PRIVATE KEY'))
         {
         Write-Host "`t`t$PathToKeyFile is in PKCS1 format" -ForegroundColor Green
         return $true
         }
     Else
         {
         Write-Host "`t`t$PathToKeyFile is not in PKCS1 format" -ForegroundColor Red
         return $false
         }
 }
 
 #---------------
 #---------------
 Function New-CertificateSigningRequest
 {
     Param (
         [string]$ConfigFileName,
         [string]$KeyFileName,
         [string]$CSRFileName
         )
 
     $strPathToKeyFile = "$CertStorePath\$KeyFileName"
     $strPathToCSRFile = "$CertStorePath\$CSRFileName"
     $strPathToConfigFile = "$CertStorePath\$ConfigFileName"
 
     Write-Host "`tCreating the certificate signing request"
     Start-Process $Global:strOpenSSLExecutablePath -ArgumentList " req -new -nodes -out `"$strPathToCSRFile`" -keyout `"$strPathToKeyFile`" -config `"$strPathToConfigFile`"" -WorkingDirectory $Global:strWorkingDirectoryPath -Wait
 
     If (Test-PKCS1Key -PathToKeyFile $strPathToKeyFile)
         {
         Start-Process $Global:strOpenSSLExecutablePath -ArgumentList " req -text -noout -in `"$strPathToCSRFile`""  -WorkingDirectory $Global:strWorkingDirectoryPath -Wait
         }
 }
 
 #---------------
 #---------------
 Function Test-PEMCertificate
 {
     Param (
         [string]$PathToCRTFile
         )
 
     Write-Host "`tChecking if the certificate is in PEM format..."
 
     If ((Get-Content "$PathToCRTFile" | select -First 1).Contains('BEGIN CERTIFICATE'))
         {
         Write-Host "`t`t$PathToCRTFile is in PEM format" -ForegroundColor Green
         return $true
         }
     Else
         {
         Write-Host "`t`t$PathToCRTFile is not in PEM format" -ForegroundColor Red
         return $false
         }
 }
 
 #---------------
 #---------------
 Function Create-CAwithKey
 {
      Param (
         [string]$OrganizationName,
         [string]$OrganizationalUnitName,
         [string]$FQDN = $Global:hshDefaultValues.Item("FQDN"),
         [string]$CountryCode = $Global:hshDefaultValues.Item("CountryCode"),
         [string]$StateOrProvinceName = $Global:hshDefaultValues.Item("StateOrProvinceName"),
         [string]$LocalityCity = $Global:hshDefaultValues.Item("LocalityCity"),
         [string]$Passphrase = $Global:hshDefaultValues.Item("Passphrase"),
         [int]$NumberOfBits = $Global:hshDefaultValues.Item("DaysToExpiry"),
         [int]$NumOfDays = $Global:hshDefaultValues.Item("DaysToExpiry")
          )
 
     $strCertStorePath = (Create-CertStore -RootFolderPath $RootFolderPath -CAonly -Folder "ca")
     $strKeyFileName = "ca.key"
     $strConfigFileName = "ca.cfg"
     $strCRTFileName = "ca.crt"
     $strPathToKeyFile = "$strCertStorePath\$strKeyFileName"
     $PathToCRTFile = "$strCertStorePath\$strCRTFileName"
     $strPathToConfigFile = "$strCertStorePath\$strConfigFileName"
 
     Create-OpenSSLConfigFile `
         -CAStorePath $strCertStorePath `
         -CertStorePath $strCertStorePath `
         -NumberOfBits $NumberOfBits `
         -KeyFileName $strKeyFileName `
         -hostname $Hostname `
         -commonName $Hostname `
         -IPaddress "0.0.0.0" `
         -FQDN $FQDN `
         -CountryCode $CountryCode `
         -StateOrProvinceName $StateOrProvinceName `
         -LocalityCity $LocalityCity `
         -organizationName $OrganizationName `
         -organizationalUnitName "Certification Authorities" `
         -ConfigFileName $strConfigFileName
 
     Start-Process $Global:strOpenSSLExecutablePath -ArgumentList " genrsa -out `"$strPathToKeyFile`" $NumberOfBits" -WorkingDirectory $Global:strWorkingDirectoryPath -Wait #
 
     If (Test-PKCS1Key -PathToKeyFile $strPathToKeyFile)
         {
         Write-Host "Creating the CA's certificate and key"
         Start-Process $Global:strOpenSSLExecutablePath -ArgumentList " req -new -x509 -extensions v3_ca -days $NumOfDays -key `"$strPathToKeyFile`" -out `"$PathToCRTFile`" -config `"$strPathToConfigFile`" -outform PEM" -WorkingDirectory $Global:strWorkingDirectoryPath
         }
     Else
         {
         Write-Host "The CA certificate and key could not be created" -ForegroundColor Red
         }
 }
 
 #---------------
 #---------------
 Function Sign-CertificateSigningRequests
 {
     Param (
         [string]$ConfigFileName, #= "vpxd.cfg",
         [string]$CSRFileName, #= "vpxd.csr",
         [string]$CRTFileName,
         [string]$PathToCAKeyFile = $Global:hshDefaultValues.Item("RootFolderPath")+"\ca\ca.key",
         [string]$PathToCACRTFile = $Global:hshDefaultValues.Item("RootFolderPath")+"\ca\ca.crt"
         )
 
         $strPathToCSRFile = "$CertStorePath\$CSRFileName"
         $strPathToConfigFile = "$CertStorePath\$ConfigFileName"
         $strPathToCRTFile = "$CertStorePath\$CRTFileName"
 
         Start-Process $Global:strOpenSSLExecutablePath " ca -batch -extensions v3_req -config `"$strPathToConfigFile`" -out `"$strPathToCRTFile`" -infiles `"$strPathToCSRFile`"" -WorkingDirectory $Global:strWorkingDirectoryPath -Wait | Out-Null
         Start-Process $Global:strOpenSSLExecutablePath " x509 -in `"$strPathToCRTFile`" -out `"$strPathToCRTFile`" -outform PEM" -WorkingDirectory $Global:strWorkingDirectoryPath -Wait | Out-Null
         If (Test-PEMCertificate -PathToCRTFile $strPathToCRTFile)
             {
             return $strPathToCRTFile 
             }
 }
 
 #---------------
 #---------------
 Function Chain-Certificates
 {
     Param (
         [string]$PathToCACRTFile,
         [string]$PathToServerCRTFile,
         [string]$CertStorePath
         )
 
     Write-Host "`tCreating a certificate chain"
     $CAChain += Get-Content $PathToServerCRTFile
     $CAChain += Get-Content $PathToCACRTFile
     Set-Content -Path "$CertStorePath\chain.pem" -Value $CAChain
     return "chain.pem"
 }
 
 #---------------
 #---------------
 Function Create-PFXCertificate
     {
     Param (
         [string]$RootFolderPath,
         [string]$ServerName,
         [string]$PFXFileName,
         [string]$ChainFileName,
         [string]$KeyFileName,
         [string]$PEMFileName,
         [string]$Passphrase
         )
     $strPathToPFXFile = "$RootFolderPath\ssl\$ServerName\$PFXFileName"
     $strPathToKeyFile = "$RootFolderPath\ssl\$ServerName\$KeyFileName"
     $strPathToChainFile = "$RootFolderPath\ssl\$ServerName\$ChainFileName"
     $strPathToPEMFile = "$RootFolderPath\ssl\$ServerName\$PEMFileName"
 
     Write-Host "`tCreating the PFX File $strPathToPFXFile"
     Start-Process $Global:strOpenSSLExecutablePath " pkcs12 -in `"$strPathToPFXFile`" -nodes -out `"$PEMFileName`"" -WorkingDirectory $Global:strWorkingDirectoryPath -Wait | Out-Null
     Write-Host "To create a PEM file for vCOPS, copy and paste the text below to the command prompt using no password`r`n$Global:strOpenSSLExecutablePath pkcs12 -in `"$strPathToPFXFile`" -nodes -out `"$strPathToPEMFile`"" -ForegroundColor DarkYellow
     Start-Process $Global:strOpenSSLExecutablePath " pkcs12 -export -out `"$strPathToPFXFile`" -in `"$strPathToChainFile`" -inkey `"$strPathToKeyFile`" -name `"$ServerName created with Andre's Certificator`" -passout pass:$Passphrase" -WorkingDirectory $Global:strWorkingDirectoryPath -Wait | Out-Null
     return $strPathToPFXFile
     }
 
 #---------------
 #---------------
 Function New-VCSA55Certificates
 {
     Param (
         [string]$OrganizationName,
         [string]$RootFolderPath,
         [string]$hostname,
         [string]$IPaddress,
         [string]$FQDN,
         [string]$CountryCode,
         [string]$StateOrProvinceName,
         [string]$LocalityCity,
         [string]$CAStorePath,
         [string]$PathToCAKeyFile,
         [string]$PathToCACRTFile,
         [string]$Password
         )
 
     #-----------------------------------------------------------------------------#
         $strCAKeyPass = "testpassword"                                            #
         [int]$NumberOfBits = 2048                                                 #
         $strKeyFileName = "rui.key"                                               #
     #-----------------------------------------------------------------------------#
 
     @("vpxd","inventoryservice","autodeploy","logbrowser") | foreach {
         Write-Host "Creating $_ files"
         $strCertStorePath = (Create-CertStore -RootFolderPath $RootFolderPath -Folder $_)
         $strConfigFileName = "$_.cfg"
         $strCSRFileName = "$_.csr"
         Switch -exact ($_)
             {
             "vpxd" 
                 {
                 $strOrganizationalUnitName = "VMware vSphere Autodeploy Service Certificate"
                 }
             "inventoryservice" 
                 {
                 $strOrganizationalUnitName = "VMware LogBrowser Service Certificate"
                 }
             "autodeploy" 
                 {
                 $strOrganizationalUnitName = "VMware Inventory Service Certificate"
                 }
             "logbrowser" 
                 {
                 $strOrganizationalUnitName = "VMware vCenter Service Certificate"
                 }
             }
         Create-OpenSSLConfigFile `
             -CertStorePath $strCertStorePath `
             -ConfigFileName "$strConfigFileName" `
             -KeyFileName "$strKeyFileName" `
             -NumberOfBits $NumberOfBits `
             -KeyPass $strCAKeyPass `
             -hostname $hostname `
             -IPaddress $IPaddress `
             -FQDN $FQDN `
             -CountryCode $CountryCode `
             -StateOrProvinceName $StateOrProvinceName `
             -LocalityCity $LocalityCity `
             -organizationName $organizationName `
             -organizationalUnitName $strOrganizationalUnitName `
             -CAStorePath $CAStorePath
         New-CertificateSigningRequest `
             -CertStorePath $strCertStorePath `
             -ConfigFileName $strConfigFileName `
             -KeyFileName $strKeyFileName `
             -CSRFileName $strCSRFileName
         $strPathToCRTFile = Sign-CertificateSigningRequests `
             -CertStorePath $strCertStorePath `
             -ConfigFileName $strConfigFileName `
             -CSRFileName $strCSRFileName `
             -CRTFileName "rui.crt" `
             -PathToCAKeyFile $PathToCAKeyFile `
             -PathToCACRTFile $PathToCACRTFile
         Chain-Certificates `
             -PathToCACRTFile $PathToCACRTFile `
             -PathToServerCRTFile $strPathToCRTFile `
             -CertStorePath $strCertStorePath
         }
 }
 
 #---------------
 #---------------
 Function Form-ESXiHostsAndIPsHash
 {
     [void] [System.Reflection.Assembly]::LoadWithPartialName("System.Drawing")
     [void] [System.Reflection.Assembly]::LoadWithPartialName("System.Windows.Forms") 
 
     [hashtable]$Global:hshHostAndIP = $null
     $Global:hshHostAndIP = @{}
     $objWShell = New-Object -ComObject Wscript.Shell
     $Global:listHostAndIP = @()
 
     #---------------
     $objESXiHostsAndIPsHashForm = New-Object System.Windows.Forms.Form 
 
         [int]$formHeight = 420
         [int]$formWidth = 240
         [int]$lblWidth = 80
         [int]$lblHeight = 20
         [int]$tbWidth = 120
         [int]$tbHeight = 20
         [int]$gap = 4
 
         $objESXiHostsAndIPsHashForm.Text = "Certificator"
         $objESXiHostsAndIPsHashForm.Size = New-Object System.Drawing.Size($formWidth,$formHeight) 
         $objESXiHostsAndIPsHashForm.StartPosition = "CenterScreen"
 
         $objESXiHostsAndIPsHashForm.KeyPreview = $True
         $objESXiHostsAndIPsHashForm.Add_KeyDown({
             If ($_.KeyCode -eq "Escape")
                 {
                 $objESXiHostsAndIPsHashForm.Close()
                 }
             })
 
         #---------------
         $lblIntroduction = New-Object System.Windows.Forms.Label
 
             $lblIntroduction.Location = New-Object System.Drawing.Size(10,10)
             $lblIntroduction.Width = $formWidth - 20
             $lblIntroduction.Height = 30 
             $lblIntroduction.Text = "Enter the Hostname and IP address then click Add to populate the list" 
 
         $objESXiHostsAndIPsHashForm.Controls.Add($lblIntroduction)
 
         #---------------
         #@"
         $lblhostname = New-Object System.Windows.Forms.Label
         $tbhostname = New-Object System.Windows.Forms.TextBox
 
             $lblhostname.Size = New-Object System.Drawing.Size($lblWidth,$lblHeight)
             $lblhostname.Top = $lblIntroduction.Bottom + $gap
             $lblhostname.Left = $lblIntroduction.Left
             $lblhostname.Text = "Hostname"
 
             $tbhostname.Size = New-Object System.Drawing.Size($tbWidth,$tbHeight)
             $tbhostname.Top = $lblhostname.Top
             $tbhostname.Left = $lblhostname.Right + 10
             $tbhostname.Text = ""
 
         $objESXiHostsAndIPsHashForm.Controls.Add($lblhostname) 
         $objESXiHostsAndIPsHashForm.Controls.Add($tbhostname) 
         #"@
 
         #---------------
         #@"
         $lblIPaddress = New-Object System.Windows.Forms.Label
         $tbIPaddress = New-Object System.Windows.Forms.TextBox
 
             $lblIPaddress.Size = New-Object System.Drawing.Size($lblWidth,$lblHeight)
             $lblIPaddress.Top = $lblhostname.Bottom + $gap
             $lblIPaddress.Left = $lblIntroduction.Left
             $lblIPaddress.Text = "IP Address"
 
             $tbIPaddress.Size = New-Object System.Drawing.Size($tbWidth,$tbHeight)
             $tbIPaddress.Top = $lblIPaddress.Top
             $tbIPaddress.Left = $lblIPaddress.Right + 10
             $tbIPaddress.Text = ""
 
         $objESXiHostsAndIPsHashForm.Controls.Add($lblIPaddress) 
         $objESXiHostsAndIPsHashForm.Controls.Add($tbIPaddress) 
         #"@
 
         #---------------
         $btnAdd = New-Object System.Windows.Forms.Button
 
             $btnAdd.Top = $lblIPaddress.Bottom + 10
             $btnAdd.Left = $lblIntroduction.Left
             $btnAdd.Width = 200
             $btnAdd.Height = 20
             $btnAdd.Text = "Add"
 
             $btnAdd.Add_Click({
                 If ($Global:hshHostAndIP.Contains($tbhostname.Text))
                     {
                     $strPopupMessage = "Entry already exists.`r`nYou already added " + $tbhostname.Text + " to the list.`r`nPlease enter a unique value. "
                     $objWShell.Popup($strPopupMessage,0,"Done",0x1)
                     }
                 Else
                     {
                     $Global:hshHostAndIP.Add($tbhostname.Text,$tbIPaddress.Text)
                     $lbHostsAndIPs.Text += $tbhostname.Text + "," + $tbIPaddress.Text +"`r`n"
                     }
                 })
 
         $objESXiHostsAndIPsHashForm.Controls.Add($btnAdd)
 
         #---------------
         $lbHostsAndIPs = New-Object System.Windows.Forms.Label
 
             $lbHostsAndIPs.Top = $btnAdd.Bottom + 10
             $lbHostsAndIPs.Left = $lblIntroduction.Left
             $lbHostsAndIPs.Width = 200
             $lbHostsAndIPs.Height = 200
             $lbHostsAndIPs.Text = ""
 
         $objESXiHostsAndIPsHashForm.Controls.Add($lbHostsAndIPs)
 
 
         #---------------
         $btnDone = New-Object System.Windows.Forms.Button
 
             $btnDone.Top = $lbHostsAndIPs.Bottom + 10
             $btnDone.Left = $lblIntroduction.Left
             $btnDone.Width = 200
             $btnDone.Height = 40
             $btnDone.Text = "Done"
 
             $btnDone.Add_Click(
                 {
                 $objESXiHostsAndIPsHashForm.Close()
                 })
 
         $objESXiHostsAndIPsHashForm.Controls.Add($btnDone)
 
     $objESXiHostsAndIPsHashForm.Add_Shown({$objESXiHostsAndIPsHashForm.Activate()})
     [void] $objESXiHostsAndIPsHashForm.ShowDialog()
 }
 
 #---------------
 #---------------
 Function New-ESXi55Certificates
 {
     Param (
         [string]$OrganizationName,
         [string]$RootFolderPath,
         [string]$FQDN,
         [string]$CountryCode,
         [string]$StateOrProvinceName,
         [string]$LocalityCity,
         [string]$CAStorePath,
         [string]$PathToCAKeyFile,
         [string]$PathToCACRTFile
         )
 
     #-----------------------------------------------------------------------------#
         $strCAKeyPass = "testpassword"                                            #
         [int]$NumberOfBits = 2048                                                 #
         $strKeyFileName = "rui.key"                                               #
         $strCSRFileName = 'rui.csr'                                               #
     #-----------------------------------------------------------------------------#
 
     $Global:hshHostAndIP.GetEnumerator() | foreach {
         Write-Host "Creating "$_.Name" files"
         $strCertStorePath = (Create-CertStore -RootFolderPath $RootFolderPath -Folder $_.Name)
         $strConfigFileName = $_.Name +'.cfg'
         $strKeyFileName = "rui.key"
         $strCSRFileName = $_.Name +'.csr'
         $strIPAddress = $Global:hshHostAndIP.Get_Item($_.Name)
         $strCommonName = ""
         Create-OpenSSLConfigFile `
             -CAStorePath $CAStorePath `
             -CertStorePath $strCertStorePath `
             -NumberOfBits $NumberOfBits `
             -KeyFileName "$strKeyFileName" `
             -KeyPass $strCAKeyPass `
             -hostname $_.Name `
             -IPaddress $strIPAddress `
             -FQDN $FQDN `
             -CountryCode $CountryCode `
             -StateOrProvinceName $StateOrProvinceName `
             -LocalityCity $LocalityCity `
             -organizationName $OrganizationName `
             -organizationalUnitName "ESXi Hosts" `
             -ConfigFileName "$strConfigFileName"
         New-CertificateSigningRequest `
             -CertStorePath $strCertStorePath `
             -ConfigFileName $strConfigFileName `
             -KeyFileName $strKeyFileName `
             -CSRFileName $strCSRFileName
         $strPathToCRTFile = Sign-CertificateSigningRequests `
             -CertStorePath $strCertStorePath `
             -ConfigFileName $strConfigFileName `
             -CSRFileName $strCSRFileName `
             -CRTFileName "rui.crt" `
             -PathToCAKeyFile $PathToCAKeyFile `
             -PathToCACRTFile $PathToCACRTFile
         Chain-Certificates `
             -PathToCACRTFile $PathToCACRTFile `
             -PathToServerCRTFile $strPathToCRTFile `
             -CertStorePath $strCertStorePath
         }
 }
 
 #---------------
 #---------------
 Function New-ServerCertificate
 {
     Param (
         [string]$OrganizationName,
         [string]$ServerName,
         [string]$ServerIP,
         [string]$RootFolderPath,
         [string]$FQDN,
         [string]$CountryCode,
         [string]$StateOrProvinceName,
         [string]$LocalityCity,
         [string]$Passphrase,
         [string]$NumberOfBits,
         [string]$CAStorePath,
         [string]$PathToCAKeyFile,
         [string]$PathToCACRTFile
         )
 
     Write-Host "Creating $ServerName files"
     $strCertStorePath = (Create-CertStore -RootFolderPath $RootFolderPath -Folder $ServerName)
     $strConfigFileName = $ServerName +'.cfg'
     $strKeyFileName = $ServerName + '.key'
     $strCSRFileName = $ServerName +'.csr'
     $strIPAddress = $ServerIP
     $strCommonName = $ServerName + "." + $FQDN
     Create-OpenSSLConfigFile `
         -CAStorePath $CAStorePath `
         -CertStorePath $strCertStorePath `
         -NumberOfBits $NumberOfBits `
         -KeyFileName "$strKeyFileName" `
         -KeyPass $strCAKeyPass `
         -hostname $ServerName `
         -IPaddress $strIPAddress `
         -FQDN $FQDN `
         -CountryCode $CountryCode `
         -StateOrProvinceName $StateOrProvinceName `
         -LocalityCity $LocalityCity `
         -organizationName $OrganizationName `
         -organizationalUnitName "Web Servers" `
         -ConfigFileName "$strConfigFileName"
     New-CertificateSigningRequest `
         -CertStorePath $strCertStorePath `
         -ConfigFileName $strConfigFileName `
         -KeyFileName $strKeyFileName `
         -CSRFileName $strCSRFileName
     $strPathToCRTFile = Sign-CertificateSigningRequests `
         -CertStorePath $strCertStorePath `
         -ConfigFileName $strConfigFileName `
         -CSRFileName $strCSRFileName `
         -CRTFileName "$ServerName.crt"`
         -PathToCAKeyFile $PathToCAKeyFile `
         -PathToCACRTFile $PathToCACRTFile
     $strChainFileName = Chain-Certificates `
         -PathToCACRTFile $PathToCACRTFile `
         -PathToServerCRTFile $strPathToCRTFile `
         -CertStorePath $strCertStorePath
     $Global:tbPFXFile.Text = Create-PFXCertificate `
         -RootFolderPath $RootFolderPath `
         -ServerName $ServerName `
         -PFXFileName "$ServerName.pfx" `
         -ChainFileName $strChainFileName `
         -KeyFileName "$ServerName.key" `
         -PEMFileName "$ServerName.pem" `
         -Passphrase $Passphrase
 }
 
 #---------------
 #---------------
 Function Upload-vCenterCertificates
     {
     Param
         (
         [string]$SSOAdminPassword,
         [string]$LookupServiceIP,
         [string]$rootPassword,
         [string]$RootFolderPath        
         )
 
     $strIPandPort = $LookupServiceIP+':7444'
     @("ca","vpxd","inventoryservice","autodeploy","logbrowser") | foreach {
         $strUploadCommand = `
 @"
  /console /command "open root:$rootPassword@$LookupServiceIP -hostkey=*" "call mkdir /ssl" "put ""$RootFolderPath\ssl\$_"" /ssl/" "exit"
 "@
         Start-Process $Global:strWinSCPExecutablePath -ArgumentList $strUploadCommand -Wait
         }
         $strCertCommands = `
 @" 
   /console /command "open root:$rootPassword@$LookupServiceIP -hostkey=*" "call service vmware-stsd stop" "call service vmware-vpxd stop" "cd /ssl/vpxd/" "call /usr/sbin/vpxd_servicecfg certificate change chain.pem rui.key" "call service vmware-stsd start" "call cd /etc/vmware-sso/register-hooks.d" "call ./02-inventoryservice --mode uninstall --ls-server https://$strIPandPort/lookupservice/sdk" "call cd /ssl/inventoryservice" "call openssl pkcs12 -export -out rui.pfx -in chain.pem -inkey rui.key -name rui -passout pass:testpassword" "call cp rui.key /usr/lib/vmware-vpx/inventoryservice/ssl" "call cp rui.crt /usr/lib/vmware-vpx/inventoryservice/ssl" "call cp rui.pfx /usr/lib/vmware-vpx/inventoryservice/ssl" "call cd /usr/lib/vmware-vpx/inventoryservice/ssl/" "call chmod 400 rui.key rui.pfx" "call chmod 644 rui.crt" "call cd /etc/vmware-sso/register-hooks.d" "call ./02-inventoryservice --mode install --ls-server https://$strIPandPort/lookupservice/sdk --user administrator@vsphere.local --password $SSOAdminPassword" "call rm /var/vmware/vpxd/inventoryservice_registered" "call service vmware-inventoryservice stop" "call service vmware-vpxd stop" "call service vmware-inventoryservice start" "call service vmware-vpxd start" "call cd /etc/vmware-sso/register-hooks.d" "call ./09-vmware-logbrowser --mode uninstall --ls-server https://$strIPandPort/lookupservice/sdk" "call cd /ssl/logbrowser" "call openssl pkcs12 -export -in rui.crt -inkey rui.key -name rui -passout pass:testpassword -out rui.pfx" "call cp rui.key /usr/lib/vmware-logbrowser/conf" "call cp rui.crt /usr/lib/vmware-logbrowser/conf" "call cp rui.pfx /usr/lib/vmware-logbrowser/conf" "call cd /usr/lib/vmware-logbrowser/conf" "call chmod 400 rui.key rui.pfx" "call chmod 644 rui.crt" "call cd /etc/vmware-sso/register-hooks.d" "call ./09-vmware-logbrowser --mode install --ls-server https://$strIPandPort/lookupservice/sdk --user administrator@vsphere.local --password $SSOAdminPassword" "call service vmware-logbrowser stop" "call service vmware-logbrowser start" "call cp /ssl/autodeploy/rui.crt /etc/vmware-rbd/ssl/waiter.crt" "call cp /ssl/autodeploy/rui.key /etc/vmware-rbd/ssl/waiter.key" "call cd /etc/vmware-rbd/ssl/" "call chmod 644 waiter.crt" "call chmod 400 waiter.key" "call chown deploy:deploy waiter.crt waiter.key" "call service vmware-rbd-watchdog stop" "call rm /var/vmware/vpxd/autodeploy_registered" "call service vmware-vpxd restart" "exit"
 "@
     Start-Process $Global:strWinSCPExecutablePath -ArgumentList $strCertCommands -Wait
 }
 
 #---------------
 #---------------
 Function Upload-HostCertificates
 	{
 	Param
 		(
 		[string]$RootFolderPath,
         [string]$rootPassword
 		)
 	    $Global:hshHostAndIP.GetEnumerator() | foreach
 		    {
             $strIPAddress = $Global:hshHostAndIP.Get_Item($_.Name)
             $strCertStorePath = "$RootFolderPath\ssl\$_.Name"
         $strUploadCommand = `
 @"
 "/console /command "open root:$rootPassword@$strIPAddress -hostkey=*" "call cd  /etc/vmware/ssl" "call mkdir backup" "call mv *.* ./backup" "call rm rui.crt" "call rm rui.key" "put ""$strCertStorePath"" /etc/vmware/ssl" "exit"
 "@
 
     Start-Process $Global:strWinSCPExecutablePath -ArgumentList $strUploadCommand -Wait
             }
 	}
 
 #---------------
 #---------------
 Function Get-CertificatorDefaults
 	{
     [void] [System.Reflection.Assembly]::LoadWithPartialName("System.Drawing")
     [void] [System.Reflection.Assembly]::LoadWithPartialName("System.Windows.Forms") 
 
     #---------------
     $objDefaultsList = New-Object System.Windows.Forms.Form 
 
         $objDefaultsList.KeyPreview = $True
 
         [int]$gap = 4
         $objDefaultsList.Text = "Certificator"
         $objDefaultsList.AutoSize = $True
         $objDefaultsList.StartPosition = "CenterScreen"
 
         #---------------
         $lblIntroduction = New-Object System.Windows.Forms.Label
 
             $lblIntroduction.Location = New-Object System.Drawing.Size(10,10)
             $lblIntroduction.Height = 50
             $lblIntroduction.Width = 300
             $lblIntroduction.Text = "Previously saved default files were found.`r`nSelect the file you want to use and click OK.`r`nClick Cancel to use the script defaults." 
 
         $objDefaultsList.Controls.Add($lblIntroduction) 
 
         #---------------
         $hdrFileName = New-Object System.Windows.Forms.ColumnHeader
         $hdrCompany = New-Object System.Windows.Forms.ColumnHeader
         $lstDefaultsFiles = New-Object System.Windows.Forms.ListView
         $btnCancel = New-Object System.Windows.Forms.Button
         $btnOK = New-Object System.Windows.Forms.Button 
 
             $hdrFileName.Text = "File Name"
             $hdrFileName.TextAlign = [System.Windows.Forms.HorizontalAlignment]::Left
 
             $hdrCompany.Text = "Company"
             $hdrCompany.TextAlign = [System.Windows.Forms.HorizontalAlignment]::Center
 
             $lstDefaultsFiles.Top = $lblIntroduction.Bottom + $gap
             $lstDefaultsFiles.Left = $lblIntroduction.Left
             $lstDefaultsFiles.Height = 200
             $lstDefaultsFiles.Width = 200
             $lstDefaultsFiles.GridLines = $true
             $lstDefaultsFiles.Columns.Add($hdrFileName.Text, -2)
             $lstDefaultsFiles.Columns.Add($hdrCompany.Text, -2)
             $lstDefaultsFiles.View = [System.Windows.Forms.View]::Details
             $lstDefaultsFiles.FullRowSelect = $True
 
             Get-ChildItem $env:APPDATA\Certificator | foreach {
 				$arrFileName = $_.ToString().Split(".")
                 $lvItem = New-Object System.Windows.Forms.ListViewItem($_.Name)
                 $lvItem.SubItems.Add($arrFileName[0])
                 $lstDefaultsFiles.Items.Add($lvItem)
                 }
 
             $btnOK.Height = 20
             $btnOK.Width = ($lstDefaultsFiles.Width/2)-5
             $btnOK.Top = $lstDefaultsFiles.Bottom + $gap
             $btnOK.Left = $lblIntroduction.Left
             $btnOK.Text = "OK"
 
             $btnCancel.Height = $btnOK.Height
             $btnCancel.Width = $btnOK.Width
             $btnCancel.Top = $btnOK.Top
             $btnCancel.Left = $btnOK.Right + 10
             $btnCancel.Text = "Cancel"
 
             $btnOK.Add_Click({
                 $lstDefaultsFiles.SelectedItems | foreach {
                     $strSelectedFile = $_.Text
                     $Global:hshDefaultValues = Import-Clixml "$env:APPDATA\Certificator\$strSelectedFile"
                     $objDefaultsList.Close()
                     }
                 })
 
             $btnCancel.Add_Click({
                 $objDefaultsList.Close()
                 })
 
         $objDefaultsList.Controls.Add($btnCancel) 
         $objDefaultsList.Controls.Add($btnOK) 
         $objDefaultsList.Controls.Add($lstDefaultsFiles) 
 
     $objDefaultsList.Add_KeyDown({
         If ($_.KeyCode -eq "Escape")
             {
             $objDefaultsList.Close()
             }
         })
 
     $objDefaultsList.Add_Shown({$objDefaultsList.Activate()})
     [void] $objDefaultsList.ShowDialog()
     }
 
 #---------------
 #---------------
 Function Form-CertificateInfo
 {
     Param (
         $Source
         )
 
     #---------------
     $objShell = New-Object -ComObject shell.application
     $arrOpenSSLCheckResult = @()
 
     [int]$formHeight = 650
     [int]$formWidth = 510
     [int]$tbHeight = 20
     [int]$tbWidth = 260
     [int]$lblHeight = 31
     [int]$lblWidth = 150
     [int]$gap = 4
 
     [void] [System.Reflection.Assembly]::LoadWithPartialName("System.Drawing") 
     [void] [System.Reflection.Assembly]::LoadWithPartialName("System.Windows.Forms") 
 
     #---------------
     $objForm = New-Object System.Windows.Forms.Form 
 
         $objForm.Text = "Certificator"
         $objForm.AutoSize = $True
         $objForm.StartPosition = "CenterScreen"
 
         #---------------
         $lblIntroduction = New-Object System.Windows.Forms.Label
 
             $lblIntroduction.Location = New-Object System.Drawing.Size(10,10)
             $lblIntroduction.Width = $objForm.Width-300
             $lblIntroduction.Height = 30 
             $lblIntroduction.Text = "" 
 
         $objForm.Controls.Add($lblIntroduction) 
 
         #---------------
         $lblCompanyName = New-Object System.Windows.Forms.Label
         $tbCompanyName_ws = New-Object System.Windows.Forms.TextBox 
 
             $lblCompanyName.Size = New-Object System.Drawing.Size($lblWidth,$lblHeight)
             $lblCompanyName.Top = $lblIntroduction.Bottom + 10
             $lblCompanyName.Left = $lblIntroduction.Left
             $lblCompanyName.Text = "Company Name"
 
             $tbCompanyName_ws.Size = New-Object System.Drawing.Size($tbWidth,$tbHeight) 
             $tbCompanyName_ws.Top = $lblCompanyName.Top
             $tbCompanyName_ws.Left = $lblCompanyName.Right + 10
             $tbCompanyName_ws.Text = $Global:hshDefaultValues.Item("Company")
 
         $objForm.Controls.Add($lblCompanyName) 
         $objForm.Controls.Add($tbCompanyName_ws) 
 
         #---------------
         $btnRootFolderBrowse = New-Object System.Windows.Forms.Button
         $tbRootFolder = New-Object System.Windows.Forms.TextBox 
         $lblCertificateStoreCheck = New-Object System.Windows.Forms.Label
 
             $btnRootFolderBrowse.Size = New-Object System.Drawing.Size($lblWidth,$lblHeight)
             $btnRootFolderBrowse.Top = $lblCompanyName.Bottom + $gap
             $btnRootFolderBrowse.Left = $lblIntroduction.Left
             $btnRootFolderBrowse.Text = "Root Folder"#`r`n(Parent for the ssl folder)"
 
             $tbRootFolder.Size = New-Object System.Drawing.Size($tbWidth,$tbHeight)
             $tbRootFolder.Top = $btnRootFolderBrowse.Top + 5
             $tbRootFolder.Left = $btnRootFolderBrowse.Right + 10
             $tbRootFolder.Text = $Global:hshDefaultValues.Item("RootFolderPath")
 
             $lblCertificateStoreCheck.Width = $objForm.Width
             $lblCertificateStoreCheck.Height = $lblHeight 
             $lblCertificateStoreCheck.Top = $btnRootFolderBrowse.Bottom + 1
             $lblCertificateStoreCheck.Left = $lblIntroduction.Left
 
             $btnRootFolderBrowse.Add_Click({$tbRootFolder.Text = ($objShell.BrowseForFolder(0,"Please select the folder where the certificate store should be created:",0,$strDefaultBrowseFolderPath).self.Path)
             $arrCertStoreCheckResult = @(Check-CertStore -RootFolderPath $tbRootFolder.Text)
             If (-Not $arrCertStoreCheckResult[0])
                 {
                 $lblCertificateStoreCheck.ForeColor = "Red"
                 $lblCertificateStoreCheck.Text = "An ssl folder and the relevant subfolders don't exist in the specified location `r`nand will be created."
                 }
             ElseIf ($arrCertStoreCheckResult[0] -and $arrCertStoreCheckResult[1] -lt 5)
                 {
                 $lblCertificateStoreCheck.ForeColor = "Orange"
                 $lblCertificateStoreCheck.Text = "An ssl folder exists in the specified location, but some or all of the subfolders `r`nneed to be created."
                 }
             ElseIf ($arrCertStoreCheckResult[0] -and $arrCertStoreCheckResult[1] -eq 5)
                 {
                 $lblCertificateStoreCheck.ForeColor = "Green"
                 $lblCertificateStoreCheck.Text = "An ssl folder and all of the required subfolders exist."
                 }
             Else {$lblCertificateStoreCheck.Text = "The result is undetermined";Write-Host "Undetermined"}
                 })
 
         $objForm.Controls.Add($tbRootFolder) 
         $objForm.Controls.Add($btnRootFolderBrowse)
         $objForm.Controls.Add($lblCertificateStoreCheck) 
 
         #---------------
         $lblhostname = New-Object System.Windows.Forms.Label
         $tbhostname = New-Object System.Windows.Forms.TextBox
 
             $lblhostname.Size = New-Object System.Drawing.Size($lblWidth,$lblHeight)
             $lblhostname.Top = $lblCertificateStoreCheck.Bottom + $gap
             $lblhostname.Left = $lblIntroduction.Left
 
             $tbhostname.Size = New-Object System.Drawing.Size($tbWidth,$tbHeight)
             $tbhostname.Top = $lblhostname.Top
             $tbhostname.Left = $lblhostname.Right + 10
 
             If ($Source -eq "btnNewCA")
                 {
                 $lblhostname.Text = "Common Name"
                 $tbhostname.Text = $tbCompanyName_ws.Text + ' Root CA'
                 }
             ElseIf ($Source -eq "btnVCSACerts" -or $Source -eq "btnESXiCerts" -or $Source -eq "btnDefaults")
                 {
                 $lblhostname.Text = "vCenter Hostname"
                 $tbhostname.Text = $Global:hshDefaultValues.Item("vCenterHostname")
                 }
             Else
                 {
                 $lblhostname.Text = "Hostname"
                 $tbhostname.Text = "WEB01"
                 }
 
         $objForm.Controls.Add($lblhostname) 
         $objForm.Controls.Add($tbhostname) 
 
         If (-Not ($Source -eq "btnNewCA"))
             {
             If (-not ($Source -eq "btnESXiCerts"))
                 {
 
                 #---------------
                 $lblIPaddress = New-Object System.Windows.Forms.Label
                 $tbIPaddress = New-Object System.Windows.Forms.TextBox
 
                     $lblIPaddress.Size = New-Object System.Drawing.Size($lblWidth,$lblHeight)
                     $lblIPaddress.Top = $lblhostname.Bottom + $gap
                     $lblIPaddress.Left = $lblIntroduction.Left
                     If ($Source -eq "btnVCSACerts")
                         {
                         $lblIPaddress.Text = "vCenter IP Address"
                         $tbIPaddress.Text = $Global:hshDefaultValues.Item("vCenterIPAddress")
                         }
                     Else
                         {
                         $lblIPaddress.Text = "IP Address"
                         $tbIPaddress.Text = "10.20.30.40"
                         }
 
                     $tbIPaddress.Size = New-Object System.Drawing.Size($tbWidth,$tbHeight)
                     $tbIPaddress.Top = $lblIPaddress.Top
                     $tbIPaddress.Left = $lblIPaddress.Right + 10
 
                 $objForm.Controls.Add($lblIPaddress) 
                 $objForm.Controls.Add($tbIPaddress) 
 
                 }
                 Else
                 {
 
                 #---------------
                 $btnManHostAndIP = New-Object System.Windows.Forms.Button
                 $btnAutoHostAndIP = New-Object System.Windows.Forms.Button 
                 $hdrHostName = New-Object System.Windows.Forms.ColumnHeader
                 $hdrIPAddress = New-Object System.Windows.Forms.ColumnHeader
                 $lstHostAndIP = New-Object System.Windows.Forms.ListView
 
                     $btnManHostAndIP.Size = New-Object System.Drawing.Size($lblWidth,$lblHeight)
                     $btnManHostAndIP.Top = $lblhostname.Bottom + $gap
                     $btnManHostAndIP.Left = $lblIntroduction.Left
                     $btnManHostAndIP.Text = "Specify Hostname and IP"
 
                     $btnAutoHostAndIP.Size = New-Object System.Drawing.Size($lblWidth,$lblHeight) 
                     $btnAutoHostAndIP.Top = $btnManHostAndIP.Bottom + $gap
                     $btnAutoHostAndIP.Left = $lblIntroduction.Left
                     $btnAutoHostAndIP.Text = "Get Hostname and IP from vCenter"
 
                     $hdrHostName.Text = "Host Name"
                     $hdrHostName.TextAlign = [System.Windows.Forms.HorizontalAlignment]::Left
 
                     $hdrIPAddress.Text = "IP Address"
                     $hdrIPAddress.TextAlign = [System.Windows.Forms.HorizontalAlignment]::Center
 
                     $lstHostAndIP.Height = $btnAutoHostAndIP.Bottom - $btnManHostAndIP.Top + $gap
                     $lstHostAndIP.Width = $tbWidth
                     $lstHostAndIP.Top = $btnManHostAndIP.Top
                     $lstHostAndIP.Left = $btnManHostAndIP.Right + 10
                     $lstHostAndIP.GridLines = $true
 
                     $lstHostAndIP.Columns.Add($hdrHostName.Text, -2)
                     $lstHostAndIP.Columns.Add($hdrIPAddress.Text, -2)
                     $lstHostAndIP.View = [System.Windows.Forms.View]::Details
 
                     $btnManHostAndIP.Add_Click({
                         Form-ESXiHostsAndIPsHash
                         $i = 1
                         $Global:hshHostAndIP.GetEnumerator() | foreach {
                             $lvItem = New-Object System.Windows.Forms.ListViewItem($_.Key)
                             $lvItem.SubItems.Add($_.Value)
                             $lstHostAndIP.Items.Add($lvItem)
                             }
                         })
 
                     $btnAutoHostAndIP.Add_Click({
                         If (Test-PowerCLI)
                             {
                             $strvCenterServer = $tbhostname.Text + '.' + $tbFQDN.te
                             Connect-VIServer -Server $tbIPAddress.Text -Credential (Get-Credential -Credential root)
                             [hashtable]$Global:hshHostAndIP = $null
                             $Global:hshHostAndIP = @{}
                             Get-VMHostNetworkAdapter * | Where-Object {$_.ManagementTrafficEnabled} | foreach {
 				                $arrHostName = $_.VMHost.ToString().Split(".")
 				                $Global:hshHostAndIP.Add($arrHostName[0],$_.IP)
 				                }
                                 $Global:hshHostAndIP.GetEnumerator() | foreach {
                                 $lvItem = New-Object System.Windows.Forms.ListViewItem($_.Key)
                                 $lvItem.SubItems.Add($_.Value)
                                 $lstHostAndIP.Items.Add($lvItem)
                                 }
                             }
                         })
 
                 $objForm.Controls.Add($btnManHostAndIP) 
                 $objForm.Controls.Add($btnAutoHostAndIP) 
                 $objForm.Controls.Add($lstHostAndIP) 
                 }
 
             #---------------
             $lblFQDN = New-Object System.Windows.Forms.Label
             $tbFQDN = New-Object System.Windows.Forms.TextBox
 
                 $lblFQDN.Size = New-Object System.Drawing.Size($lblWidth,$lblHeight)
                 $lblFQDN.Left = $lblIntroduction.Left
                 If ($Source -eq "btnESXiCerts")
                     {
                     $lblFQDN.Top = $lstHostAndIP.Bottom + $gap
                     }
                 Else
                     {
                     $lblFQDN.Top = $lblIPaddress.Bottom + $gap
                     }
                 $lblFQDN.Text = "Company FQDN"
 
                 $tbFQDN.Size = New-Object System.Drawing.Size($tbWidth,$tbHeight)
                 $tbFQDN.Top = $lblFQDN.Top
                 $tbFQDN.Left = $lblFQDN.Right + 10
                 $tbFQDN.Text = $Global:hshDefaultValues.Item("FQDN")
 
             $objForm.Controls.Add($lblFQDN) 
             $objForm.Controls.Add($tbFQDN) 
             }
 
         #---------------
         $lblCountryCode = New-Object System.Windows.Forms.Label
         $tbCountryCode = New-Object System.Windows.Forms.TextBox
 
             $lblCountryCode.Size = New-Object System.Drawing.Size($lblWidth,$lblHeight)
             If ($Source -eq "btnNewCA")
                 {
                 $lblCountryCode.Top = $lblhostname.Bottom + $gap
                 }
             Else
                 {
                 $lblCountryCode.Top = $lblFQDN.Bottom + $gap
                 }
             $lblCountryCode.Left = $lblIntroduction.Left
             $lblCountryCode.Text = "Country Code`r`n(Two letter code)"
 
             $tbCountryCode.Size = New-Object System.Drawing.Size($tbWidth,$tbHeight)
             $tbCountryCode.Top = $lblCountryCode.Top
             $tbCountryCode.Left = $lblCountryCode.Right + 10
             $tbCountryCode.Text = $Global:hshDefaultValues.Item("CountryCode")
 
         $objForm.Controls.Add($lblCountryCode) 
         $objForm.Controls.Add($tbCountryCode) 
 
         #---------------
         $lblStateOrProvinceName = New-Object System.Windows.Forms.Label
         $tbStateOrProvinceName = New-Object System.Windows.Forms.TextBox
 
             $lblStateOrProvinceName.Size = New-Object System.Drawing.Size($lblWidth,$lblHeight)
             $lblStateOrProvinceName.Top = $lblCountryCode.Bottom + $gap
             $lblStateOrProvinceName.Left = $lblIntroduction.Left
             $lblStateOrProvinceName.Text = "Province"
 
             $tbStateOrProvinceName.Size = New-Object System.Drawing.Size($tbWidth,$tbHeight)
             $tbStateOrProvinceName.Top = $lblStateOrProvinceName.Top
             $tbStateOrProvinceName.Left = $lblStateOrProvinceName.Right + 10
             $tbStateOrProvinceName.Text = $Global:hshDefaultValues.Item("StateOrProvinceName")
 
         $objForm.Controls.Add($lblStateOrProvinceName) 
         $objForm.Controls.Add($tbStateOrProvinceName) 
 
         #---------------
         $lblLocalityCity = New-Object System.Windows.Forms.Label
         $tbLocalityCity = New-Object System.Windows.Forms.TextBox
 
             $lblLocalityCity.Size = New-Object System.Drawing.Size($lblWidth,$lblHeight)
             $lblLocalityCity.Top = $lblStateOrProvinceName.Bottom + $gap
             $lblLocalityCity.Left = $lblIntroduction.Left
             $lblLocalityCity.Text = "Locality/City"
 
             $tbLocalityCity.Size = New-Object System.Drawing.Size($tbWidth,$tbHeight)
             $tbLocalityCity.Top = $lblLocalityCity.Top
             $tbLocalityCity.Left = $lblLocalityCity.Right + 10
             $tbLocalityCity.Text = $Global:hshDefaultValues.Item("LocalityCity")
 
         $objForm.Controls.Add($lblLocalityCity) 
         $objForm.Controls.Add($tbLocalityCity) 
 
         #---------------
         If ($Source -eq "btnNewCA" -or $Source -eq "btnCustomCert" -or $Source -eq "btnDefaults")
             {
             $lblNumberOfBits = New-Object System.Windows.Forms.Label
             $tbNumberOfBits = New-Object System.Windows.Forms.TextBox
 
                 $lblNumberOfBits.Size = New-Object System.Drawing.Size($lblWidth,$lblHeight)
                 $lblNumberOfBits.Top = $lblLocalityCity.Bottom + $gap
                 $lblNumberOfBits.Left = $lblIntroduction.Left
                 $lblNumberOfBits.Text = "Number of Bits"
 
                 $tbNumberOfBits.Size = New-Object System.Drawing.Size($tbWidth,$tbHeight)
                 $tbNumberOfBits.Top = $lblNumberOfBits.Top
                 $tbNumberOfBits.Left = $lblNumberOfBits.Right + 10
                 $tbNumberOfBits.Text = $Global:hshDefaultValues.Item("NumberOfBits")
 
             $objForm.Controls.Add($lblNumberOfBits) 
             $objForm.Controls.Add($tbNumberOfBits) 
             }
 
         #---------------
         If ($Source -eq "btnVCSACerts" -or $Source -eq "btnCustomCert" -or $Source -eq "btnESXiCerts" -or $Source -eq "btnDefaults")
             {
             $btnSigningCAdirBrowse = New-Object System.Windows.Forms.Button
             $tbSigningCAdir = New-Object System.Windows.Forms.TextBox 
 
                 $btnSigningCAdirBrowse.Size = New-Object System.Drawing.Size($lblWidth,$lblHeight)
                 If ($Source -eq "btnCustomCert" -or $Source -eq "btnDefaults")
                     {
                     $btnSigningCAdirBrowse.Top = $lblNumberOfBits.Bottom + $gap
                     }
                 ElseIf ($Source -eq "btnESXiCerts"  -or $Source -eq "btnVCSACerts")
                     {
                     $btnSigningCAdirBrowse.Top = $lblLocalityCity.Bottom + $gap
                     }
                 $btnSigningCAdirBrowse.Left = $lblIntroduction.Left
                 $btnSigningCAdirBrowse.Text = "Signing CA Root Directory"
 
                 $tbSigningCAdir.Size = New-Object System.Drawing.Size($tbWidth,$tbHeight)
                 $tbSigningCAdir.Top = $btnSigningCAdirBrowse.Top + 5
                 $tbSigningCAdir.Left = $btnSigningCAdirBrowse.Right + 10
                 $tbSigningCAdir.Text = $Global:hshDefaultValues.Item("SigningCARoot")
 
                 $btnSigningCAdirBrowse.Add_Click({
                     $tbSigningCAdir.Text = ($objShell.BrowseForFolder(0,"Please select the folder where the Certification Authority files are stored:",0,$strDefaultBrowseFolderPath).self.Path)
                     })
 
             $objForm.Controls.Add($btnSigningCAdirBrowse)
             $objForm.Controls.Add($tbSigningCAdir) 
             }
 
             #---------------
             $btnSigningCAcrtBrowse = New-Object System.Windows.Forms.Button
             $tbSigningCAcrt = New-Object System.Windows.Forms.TextBox 
             $dlgSigningCAcrt = New-Object System.Windows.Forms.OpenFileDialog
 
                 $btnSigningCAcrtBrowse.Size = New-Object System.Drawing.Size($lblWidth,$lblHeight)
                 If ($Source -eq "btnNewCA")
                     {
                     $btnSigningCAcrtBrowse.Top = $lblNumberOfBits.Bottom + $gap
                     }
                 Else
                     {
                     $btnSigningCAcrtBrowse.Top = $btnSigningCAdirBrowse.Bottom + $gap
                     }
                 $btnSigningCAcrtBrowse.Left = $lblIntroduction.Left
                 $btnSigningCAcrtBrowse.Text = "Signing CA .crt file"
 
                 $tbSigningCAcrt.Size = New-Object System.Drawing.Size($tbWidth,$tbHeight) 
                 $tbSigningCAcrt.Top = $btnSigningCAcrtBrowse.Top + 5
                 $tbSigningCAcrt.Left = $btnSigningCAcrtBrowse.Right + 10 
                 $tbSigningCAcrt.Text = $Global:hshDefaultValues.Item("SigningCACRT")
 
                 $dlgSigningCAcrt.Title = "Specify the Certificate file for the Certification Authority" ;
                 $dlgSigningCAcrt.InitialDirectory = $Global:hshDefaultValues.Item("RootFolderPath")
                 $dlgSigningCAcrt.Filter = "Certificate files (*.crt)|*.crt|All files (*.*)|*.*"
 
                 $btnSigningCAcrtBrowse.Add_Click({
                     If ($dlgSigningCAcrt.ShowDialog() -eq "OK")
                         {
                         $tbSigningCAcrt.Text = $dlgSigningCAcrt.FileName
                         }
                     })
 
             $objForm.Controls.Add($btnSigningCAcrtBrowse)
             $objForm.Controls.Add($tbSigningCAcrt) 
 
         If ($Source -eq "btnVCSACerts" -or $Source -eq "btnCustomCert" -or $Source -eq "btnESXiCerts" -or $Source -eq "btnDefaults")
             {
             #---------------
             $btnSigningCAkeyBrowse = New-Object System.Windows.Forms.Button
             $tbSigningCAkey = New-Object System.Windows.Forms.TextBox 
             $dlgSigningCAkey = New-Object System.Windows.Forms.OpenFileDialog
 
                 $btnSigningCAkeyBrowse.Size = New-Object System.Drawing.Size($lblWidth,$lblHeight)
                 $btnSigningCAkeyBrowse.Top = $btnSigningCAcrtBrowse.Bottom + $gap
                 $btnSigningCAkeyBrowse.Left = $lblIntroduction.Left
                 $btnSigningCAkeyBrowse.Text = "Signing CA .key file"
 
                 $tbSigningCAkey.Size = New-Object System.Drawing.Size($tbWidth,$tbHeight)
                 $tbSigningCAkey.Top = $btnSigningCAkeyBrowse.Top + 5
                 $tbSigningCAkey.Left = $btnSigningCAkeyBrowse.Right + 10
                 $tbSigningCAkey.Text = $Global:hshDefaultValues.Item("SigningCAKey")
 
                 $dlgSigningCAkey.Title = "Specify the Key file for the Certification Authority" ;
                 $dlgSigningCAkey.InitialDirectory = $Global:hshDefaultValues.Item("RootFolderPath")
                 $dlgSigningCAkey.Filter = "Key files (*.key)|*.key|All files (*.*)|*.*"
 
                 $btnSigningCAkeyBrowse.Add_Click({
                     If ($dlgSigningCAkey.ShowDialog() -eq "OK")
                         {
                         $tbSigningCAkey.Text = $dlgSigningCAkey.FileName
                         }
                     })
 
             $objForm.Controls.Add($btnSigningCAkeyBrowse)
             $objForm.Controls.Add($tbSigningCAkey) 
             }
`

