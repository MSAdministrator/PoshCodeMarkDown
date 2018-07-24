---
Author: brian h madsen
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 5580
Published Date: 2014-11-11t03
Archived Date: 2014-11-23t16
---

# configure visio graphics - 

## Description

configure visio graphics services – just a small snippet for me to remember

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 Add-PSSnapin Microsoft.SharePoint.PowerShell -ErrorAction SilentlyContinue
 
 $ParamObject = new-object PSObject
 
 
 
 $ParamObject | Add-Member Noteproperty SecureApplicationProxy ((Get-SPServiceApplicationProxy | Where { $_.Displayname -Like "Secure*"})) -Force
 
 $ssApplicationProxy = Get-SPServiceApplicationProxy | Where { $_.Displayname -like "Secure*"}
 Update-SPSecureStoreMasterKey -ServiceApplicationProxy $ParamObject.SecureApplicationProxy -PassPhrase $ParamObject.PassPhrase > $null
 Start-Sleep 5
 Update-SPSecureStoreApplicationServerKey -ServiceApplicationProxy $ParamObject.SecureApplicationProxy -Passphrase $ParamObject.PassPhrase > $null
 Start-Sleep 5
 
 $svcApp = Get-SPServiceApplication | where {$_.TypeName -like "*Visio*"}
 $unattendedServiceAccountApplicationID = ($svcApp | Get-SPVisioExternalData).UnattendedServiceAccountApplicationID
 
 $ParamObject | Add-Member Noteproperty UnattendedAccountId $unattendedServiceAccountApplicationID -Force
 $ParamObject | Add-Member Noteproperty VisioServiceApplication ((Get-SPServiceApplication | Where { $_.DisplayName -like "Visio*"})) -Force
 
 if ([string]::IsNullOrEmpty($unattendedServiceAccountApplicationID)) {
     $unattendedAccount = Get-Credential $ParamObject.UnattendedAccount
 
     $name = "$($svcApp.ID)-VisioUnattendedAccount"
     Write-Host "Creating Secure Store Target Application $name..."
     $secureStoreTargetApp = New-SPSecureStoreTargetApplication -Name $name `
         -FriendlyName "Visio Services Unattended Account Target App" `
         -ApplicationType Group `
         -TimeoutInMinutes 3
 
     $groupClaim = New-SPClaimsPrincipal -Identity "nt authority\authenticated users" -IdentityType WindowsSamAccountName
     $adminPrincipal = New-SPClaimsPrincipal -Identity "$($env:userdomain)\$($env:username)" -IdentityType WindowsSamAccountName
 
     $usernameField = New-SPSecureStoreApplicationField -Name "User Name" -Type WindowsUserName -Masked:$false
     $passwordField = New-SPSecureStoreApplicationField -Name "Password" -Type WindowsPassword -Masked:$false
     $fields = $usernameField, $passwordField
 
     $secureUserName = ConvertTo-SecureString $unattendedAccount.UserName -AsPlainText -Force
     $securePassword = $unattendedAccount.Password
     $credentialValues = $secureUserName, $securePassword
 
     $subId = [Microsoft.SharePoint.SPSiteSubscriptionIdentifier]::Default
     $context = [Microsoft.SharePoint.SPServiceContext]::GetContext($svcApp.ServiceApplicationProxyGroup, $subId)
 
     $secureStoreApp = Get-SPSecureStoreApplication -ServiceContext $context -Name $name -ErrorAction SilentlyContinue
     if ($secureStoreApp -eq $null) {
         Write-Host "Creating Secure Store Application..."
         $secureStoreApp = New-SPSecureStoreApplication -ServiceContext $context `
             -TargetApplication $secureStoreTargetApp `
             -Administrator $adminPrincipal `
             -CredentialsOwnerGroup $groupClaim `
             -Fields $fields
     }
     Write-Host "Updating Secure Store Group Credential Mapping..."
     Update-SPSecureStoreGroupCredentialMapping -Identity $secureStoreApp -Values $credentialValues
     
     $ParamObject 
     
     $svcApp | Set-SPVisioExternalData -UnattendedServiceAccountApplicationID $name
 }
 
 Set-SPVisioPerformance �MaxDiagramCacheAge 10 �MaxCacheSize 5120 �MaxDiagramSize 15 �MaxRecalcDuration 60 �MinDiagramCacheAge 1 �VisioServiceApplication $ParamObject.VisioServiceApplication
 Set-SPVisioExternalData -VisioServiceApplication $ParamObject.VisioServiceApplication -UnattendedServiceAccountApplicationID $ParamObject.UnattendedAccountId
 cls
 Write-Host "Done..."
`

