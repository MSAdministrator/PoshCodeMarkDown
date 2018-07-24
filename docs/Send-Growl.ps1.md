---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 3.0
Encoding: ascii
License: cc0
PoshCode ID: 4834
Published Date: 2014-01-23t15
Archived Date: 2017-03-15t18
---

# send-growl - 

## Description

this is the powershell 2.0 -only continuation of my growl module — redesigned as a proper “module” that can be used by (many) other scripts.

## Comments



## Usage



## TODO



## module

`register-growltype`

## Code

`#
 #
 ##
 
 Set-StrictMode -Version 2
 $script:appName = "PowerGrowler"
 
 [Reflection.Assembly]::LoadFrom("$(Split-Path (gp HKCU:\Software\Growl).'(default)')\Growl.Connector.dll") | Out-Null
 if(!(Test-Path Variable:Global:PowerGrowlerNotices)) {
    $global:PowerGrowlerNotices = @{}
 }
 
 $script:PowerGrowler = New-Object "Growl.Connector.GrowlConnector"
 
 function Register-GrowlType {
 #.Synopsis
 #.Description
 #.Parameter AppName
 #.Parameter Name
 #.Parameter DisplayName
 #.Parameter Icon
 #.Parameter MachineName
 #.Parameter Priority
 #.Example
 #
 PARAM(
    [Parameter(Mandatory=$true,Position=0)]
    [String]$AppName
 ,
    [Parameter(Mandatory=$true,Position=1)]
    [ValidateScript( {!$global:PowerGrowlerNotices.Contains($AppName) -OR !$global:PowerGrowlerNotices.$AppName.Notices.ContainsKey($_)} )]
    [String]$Name
 ,
    [Parameter(Mandatory=$false,Position=5)]
    [String]$Icon = "$PSScriptRoot\default.ico"
 ,
    [Parameter(Mandatory=$false,Position=6)]
    [String]$DisplayName = $Name
 ,
    [Parameter(Mandatory=$false,Position=7)]
    [String]$MachineName
 ,
    [Parameter(Mandatory=$false)]
    [String]$AppIcon
 )
   
    [Growl.Connector.NotificationType]$Notice = $Name
    $Notice.DisplayName = $DisplayName
    $Notice.Icon = Convert-Path (Resolve-Path $Icon)
 
    if($MachineName) {
       $Notice.MachineName = $MachineName
    }
    if(!$global:PowerGrowlerNotices.Contains($AppName)) {
       $global:PowerGrowlerNotices.Add( $AppName, ([Growl.Connector.Application]$AppName) )
 
       $global:PowerGrowlerNotices.$AppName = Add-Member -input $global:PowerGrowlerNotices.$AppName -Name Notices -Type NoteProperty -Value (New-Object hashtable) -Passthru
       $global:PowerGrowlerNotices.$AppName.Icon = Convert-Path (Resolve-Path $AppIcon)
    }
    
    $global:PowerGrowlerNotices.$AppName.Notices.Add( $Name, $Notice )
 
    $script:PowerGrowler.Register( $global:PowerGrowlerNotices.$AppName , [Growl.Connector.NotificationType[]]@($global:PowerGrowlerNotices.$AppName.Notices.Values) )
 }
 
 
 function Set-GrowlPassword { 
 #.Synopsis
 #.Description
 #.Parameter Password
 #.Parameter Encryption
 #.Parameter KeyHash
 PARAM( 
    [Parameter(Mandatory=$true,Position=0)]
    [String]$Password
 ,
    [Parameter(Mandatory=$false,Position=1)]
    [ValidateSet( "AES", "DES", "RC2", "TripleDES", "PlainText" )]
    [String]$Encryption = "AES"
 ,   
    [Parameter(Mandatory=$false,Position=2)]
    [ValidateSet( "MD5", "SHA1", "SHA256", "SHA384", "SHA512" )]
    [String]$KeyHash = "SHA1"
 )   
    $script:PowerGrowler.EncryptionAlgorithm = [Growl.Connector.Cryptography+SymmetricAlgorithmType]::"$Encryption"
    $script:PowerGrowler.KeyHashAlgorithm = [Growl.Connector.Cryptography+SymmetricAlgorithmType]::"$KeyHash"
    $script:PowerGrowler.Password = $Password
 }
 
 Register-GrowlType $script:AppName "Default" -appIcon "$PsScriptRoot\default.ico"
 
 function Register-GrowlCallback {
 #.Synopsis
 #.Description
 #.Example
 #
 #
 PARAM(
 [Parameter(Mandatory=$true)]
 [Scriptblock]$Handler
 )
    Register-ObjectEvent $script:PowerGrowler NotificationCallback -Action $Handler
 }
 
 function Send-Growl {
 [CmdletBinding(DefaultParameterSetName="DataCallback")]
 #.Synopsis
 #.Description
 #.Parameter Caption
 #.Parameter Message
 #.Parameter NoticeType
 #.Parameter Icon
 #.Parameter Priority
 #.Example
 #
 #.Example
 #
 #
 PARAM (
    [Parameter(Mandatory=$true, Position=0)]
    [ValidateScript( {$global:PowerGrowlerNotices.Contains($AppName)} )]
    [string]$AppName
 ,
    [Parameter(Mandatory=$true, Position=1)][Alias("Type")]   
    [ValidateScript( {$global:PowerGrowlerNotices.$AppName.Notices.ContainsKey($_)} )]   
    [string]$NoticeType
 ,
    [Parameter(Mandatory=$true, Position=2)]
    [string]$Caption
 ,
    [Parameter(Mandatory=$true, Position=3)]
    [string]$Message
 ,
    [Parameter(Mandatory=$true, Position=4, ParameterSetName="UrlCallback")]
    [Uri]$Url
 ,  
    [Parameter(Mandatory=$true, Position=4, ParameterSetName="DataCallback")]
    [string]$CallbackData
 ,  
    [Parameter(Mandatory=$true, Position=5, ParameterSetName="DataCallback")]
    [string]$CallbackType
 ,
    [string]$Icon
 ,
    [Growl.Connector.Priority]$Priority = "Normal" 
 )
 
    $notice = New-Object Growl.Connector.Notification $appName, $NoticeType, (Get-Date).Ticks.ToString(), $caption, $Message
    
    if($Icon) { $notice.Icon = Convert-Path (Resolve-Path $Icon) }
    if($Priority) { $notice.Priority = $Priority }
    
    if($DebugPreference -gt "SilentlyContinue") { Write-Output $notice }
    if( Test-Path Variable:Local:Url ) {
       $context = new-object Growl.Connector.CallbackContext
       $context.Data = $(if(Test-Path Variable:Local:CallbackData){$CallbackData}else{$Url.ToString()})
       $context.Type = $(if(Test-Path Variable:Local:CallbackType){$CallbackType}else{"$NoticeType+Url"})
       $urlCb = new-object Growl.Connector.UrlCallbackTarget
       Write-Host $Url -Fore Cyan
       $urlCb.Url = $Url
       $context.SetUrlCallbackTarget($urlcb)
       $script:PowerGrowler.Notify($notice, $context)
    } elseif( (Test-Path Variable:Local:CallbackData) -and (Test-Path Variable:Local:CallbackType) ) {
       $context = new-object Growl.Connector.CallbackContext
       $context.Data = $CallbackData
       $context.Type = $CallbackType
       Write-Host $context.GetUrlCallbackTarget() -Fore Magenta
       $script:PowerGrowler.Notify($notice, $context)
    } else {          
       $script:PowerGrowler.Notify($notice)
    }
 }
 
 Export-ModuleMember -Function Send-Growl, Set-GrowlPassword, Register-GrowlCallback, Register-GrowlType
`

