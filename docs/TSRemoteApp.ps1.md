---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 6.1
Encoding: ascii
License: cc0
PoshCode ID: 2382
Published Date: 
Archived Date: 2010-11-30t16
---

# tsremoteapp - 

## Description

module gives functions for managing remoteapp on windows 2008 and windows 2008 r2.

## Comments



## Usage



## TODO



## function

`new-tsremoteapp`

## Code

`#
 #
 function New-TSRemoteApp {
 <#
 .SYNOPSIS
 Creates a new RemoteApp on Windows Server 2008 Terminal Server.
 .DESCRIPTION
 Creates a new RemoteApp using the supplied parameters.
 .PARAMETER Alias
 Alias for the new RemoteApp. Accepts ValueFromPipeline and ValueFromPipelineByPropertyName.
 .PARAMETER Applicationpath
 Path to the executable file for for the new RemoteApp. This file must exist before creating the new RemoteApp.
 Accepts ValueFromPipeline and ValueFromPipelineByPropertyName.
 .PARAMETER Displayname
 Displayname for the new RemoteApp. This is the application name the users will see. Accepts ValueFromPipeline and ValueFromPipelineByPropertyName.
 .PARAMETER ShowinRDWebAccess
 True or false. Determines if the RemoteApp should be visible in RD Web Access. Defaults to true if the parameter is omitted. Accepts ValueFromPipeline and ValueFromPipelineByPropertyName.
 .PARAMETER CommandlineSetting
 0 = Do not allow command-line arguments, 1 = Allow any command-line arguments (not recommended), 2 = Always use the following command-line arguments
 .PARAMETER CommandlineArguments
 Command-line  argument to be used when starting the new RemoteApp
 .EXAMPLE
 New-TSRemoteApp -Alias Notepad -Applicationpath "%windir%\system32\notepad.exe" -Displayname Notepad -ShowinRDWebAccess $false
 .EXAMPLE
 New-TSRemoteApp -Alias Calc -Applicationpath "%windir%\system32\calc.exe" -Displayname Calculator -CommandLineArgumentMode 2 -CommandlineArguments '/MyCustomParameter'
 .NOTES
 AUTHOR:    Wizarden
 LASTEDIT:  20.11.2010 
 SOURCE:    Based on Script by Jan Egil Ring and Rebuilded for Windows 2008 Server
 #>
 
 [CmdletBinding()]
     param (
         [parameter(Mandatory=$true,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
         [string]$Alias,
          
         [parameter(Mandatory=$true,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
         [string]$Applicationpath,
         [parameter(ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
         [string]$Displayname,
         [parameter(ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
         [boolean]$ShowinRDWebAccess = $true,
         [parameter(ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
         [int]$CommandlineSetting,
         [parameter(ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
         $CommandlineArguments
          
     )
 
 if ((Get-WmiObject -Namespace "root\CIMV2\TerminalServices" -Query "select * from Win32_TSPublishedApplication where alias = '$Alias'") -ne $null) {
 	Write-Warning "The application $alias already exist!"
 	return
 }
 
 $newapp = ([wmiclass]"ROOT\CIMV2\TerminalServices:Win32_TSPublishedApplication").CreateInstance()
 $newapp.Alias = $Alias
 $newapp.Path = [System.Environment]::ExpandEnvironmentVariables($Applicationpath)
 $newapp.VPath = $Applicationpath
 
 $newapp.ShowInPortal = $ShowinRDWebAccess
 
 if ($Displayname) {
 	$newapp.Name = $Displayname
 } else {
 	if (Test-Path $Applicationpath) {
 		if ((Get-Item $Applicationpath).VersionInfo.FileDescription -ne "") {
 			$newapp.Name = (Get-Item $Applicationpath).VersionInfo.FileDescription
 		} else {$newapp.Name = (Get-Item $Applicationpath).Name}
 	} else {$newapp.Name = [System.IO.Path]::GetFileNameWithoutExtension($Applicationpath)}
 }
 
 if ($CommandlineSetting -ne $null -and $CommandlineSetting -ne 2) {
 	$newapp.CommandLineSetting = $CommandlineSetting
 } else {$newapp.CommandLineSetting = 0}
 
 if ($CommandlineArguments -and ($CommandlineSetting -eq 2)) {
 	if ($CommandlineArguments.RequiredCommandline) {
 		$CommandlineArguments = $CommandlineArguments.RequiredCommandline
 	}
 	$newapp.CommandLineSetting = $CommandlineSetting
 	$newapp.RequiredCommandLine = $CommandlineArguments
 }
 
 $newapp.IconIndex = 0
 $newapp.IconPath = $Applicationpath
 
 $newapp.RDPFileContents=(Get-WmiObject -Class "Win32_TSDeploymentSettings" -Namespace "root\CIMV2\TerminalServices").DeploymentRDPSettings
 $newapp.RDPFileContents+="remoteapplicationmode:i:1"
 $newapp.RDPFileContents+="alternate shell:s:||$Alias"
 $newapp.RDPFileContents+="remoteapplicationprogram:s:||$Alias"
 $newapp.RDPFileContents+="remoteapplicationname:s:$newapp.Name"
 $newapp.RDPFileContents+="remoteapplicationcmdline:s:$newapp.CommandLineSetting"
 
 $newapp.Put() | Out-Null
 if ($?) {Write-Host "The application $alias was succesfully created" -ForegroundColor yellow}
 
 }
 
 function Remove-TSRemoteApp {
 <#
 .SYNOPSIS
 Removes the specified RemoteApp from the Windows 2008 Terminal Server.
 .DESCRIPTION
 Removes the specified RemoteApp from the Windows 2008 Terminal Server. One mandatory parameter: Alias
 .PARAMETER Alias
 The alias of the application to be removed
 .EXAMPLE
 Remove-TSRemoteApp -Alias Calc
 Removes the Calc RemoteApp.
 .EXAMPLE
 Get-TSRemoteApp | Foreach-Object {Remove-TSRemoteApp $_.Alias}
 Removes all RemoteApps.
 .NOTES
 AUTHOR:    Wizarden
 LASTEDIT:  20.11.2010 
 SOURCE:    Based on Script by Jan Egil Ring and Rebuilded for Windows 2008 Server
 #>
 
 [CmdletBinding()]
     param (
         [parameter(Mandatory=$true,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
         [string]$Alias
     )
 
 $Apps = Get-WmiObject -Class "Win32_TSPublishedApplication" -Namespace "root\CIMV2\TerminalServices" -Filter "Alias = '$Alias'"
 
 if (-not $Apps) {
 	Write-Warning "The application $alias doesn't exist!";return
 } else {
 	$Apps.Delete() | Out-Null
 	if (-not (Get-WmiObject -Class "Win32_TSPublishedApplication" -Namespace "root\CIMV2\TerminalServices" -Filter "Alias = '$Alias'")) {
 		Write-Host "The application $alias was successfully removed" -ForegroundColor yellow
 	}
 }
 
 }
 
 function Get-TSRemoteApp {
 <#
 .SYNOPSIS
 Retrieves info about the specified RemoteApp from the Windows 2008 Terminal Server.
 .DESCRIPTION
 Retrieves info about specified RemoteApp from the Windows 2008 Terminal Server. One optional parameter: Alias
 If Alias is omitted, all RemoteApps are returned.
 .PARAMETER Alias
 The alias of the application to be retirived
 .EXAMPLE
 Get-TSRemoteApp -Alias Calc
 .NOTES
 AUTHOR:    Wizarden
 LASTEDIT:  20.11.2010 
 SOURCE:    Based on Script by Jan Egil Ring and Rebuilded for Windows 2008 Server
 #>
 
 [CmdletBinding()]
     param (
         [parameter(Mandatory=$false)]
         [string]$Alias
     )
 
 if ($Alias) {
 	if (-not (Get-WmiObject -Class "Win32_TSPublishedApplication" -Namespace "root\CIMV2\TerminalServices" -Filter "Alias = '$Alias'")) 
 	{
 		Write-Warning "The application $alias doesn't exist!";return
 	}
 	Get-WmiObject -Class "Win32_TSPublishedApplication" -Namespace "root\CIMV2\TerminalServices" -Filter "Alias = '$Alias'" | Select-Object @{"Name"="Displayname";"Expression"={$_.Name}},@{"Name"="Alias";"Expression"={$_.Alias}},@{"Name"="Path";"Expression"={$_.VPath}}
 } else {
 	Get-WmiObject -Class "Win32_TSPublishedApplication" -Namespace "root\CIMV2\TerminalServices" | Select-Object @{"Name"="Displayname";"Expression"={$_.Name}},@{"Name"="Alias";"Expression"={$_.Alias}},@{"Name"="Path";"Expression"={$_.VPath}}
 }
 }
 
 function Import-TSRemoteAppsSet {
 <#
 .SYNOPSIS
 Imports all TS RemoteApps Settings from the provided TSPUB-file.
 .DESCRIPTION
 Imports all TS RemoteApps from the provided TSPUB-file. Accepts ValueFromPipeline and ValueFromPipelineByPropertyName.
 One mandatory parameter: Path
 .PARAMETER Path
 Path to the TSPUB-file to be imported. Accepts ValueFromPipeline and ValueFromPipelineByPropertyName.
 Parameter Type: Mandatory
 .PARAMETER Replace
 Deletes all existing RemoteApps and insert new from file. Enabled by Default.
 Parameter Type: Optional
 .PARAMETER TSDeploymentSettings
 Replace Terminal Server settings. Enabled by Default.
 Parameter Type: Optional
 .EXAMPLE
 Import-TSRemoteApps -Path C:\temp\RemoteApps.tspub
 Imports all RemoteApps from the specified TSPUB-file.
 .NOTES
 AUTHOR:    Wizarden
 LASTEDIT:  21.11.2010 
 #>
 
 [CmdletBinding()]
     param (
         [parameter(Mandatory=$true,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
         [string]$Path,
         [parameter(ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
         [switch]$NotReplace = $true,
         [parameter(ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
         [switch]$TSDeploymentSettings = $true
     )
 
 $ConfigFile = [xml](Get-Content $Path -Encoding UTF8)
 
 if ($Replace) {
 	Get-WmiObject -Class "Win32_TSPublishedApplication" -Namespace "root\CIMV2\TerminalServices" | ForEach-Object {$_.Delete() | Out-Null}
 }
 
 if ($TSDeploymentSettings) {
 
 	$Win32_TSPublishedApplicationList = Get-WmiObject -Namespace "root\CIMV2\TerminalServices" -Class Win32_TSPublishedApplicationList
 	$Win32_TSPublishedApplicationList.Disabled = ($ConfigFile.RemotePrograms.UseAllowList -eq "No")
 	$Win32_TSPublishedApplicationList.Put() | Out-Null
 	
 	$DeploymentSettings = Get-WmiObject -Class "Win32_TSDeploymentSettings" -Namespace "root\CIMV2\TerminalServices"
 
 	$DeploymentRDPSettings = $DeploymentSettings.DeploymentRDPSettings.Split("`n")
 	$values = @{}
 	
 	ForEach ($val in $DeploymentRDPSettings) {
 		if ($val -ne "") {$values += @{([regex]::Match($val,"(^.+?:\w?):(.*)")).groups[1].value = ([regex]::Match($val,"(^.+?:\w?):(.*)")).groups[2].value}}
 	}
 	
 	$values["server port:i"] = $ConfigFile.RemotePrograms.DeploymentSettings.Port
 	$values["full address:s"] = $ConfigFile.RemotePrograms.DeploymentSettings.FarmName
 	$values["gatewayhostname:s"] = $ConfigFile.RemotePrograms.DeploymentSettings.GatewaySettings.GatewayName
 	$values["gatewaycredentialssource:i"] = $ConfigFile.RemotePrograms.DeploymentSettings.GatewaySettings.GatewayAuthMode
 	$values["promptcredentialonce:i"] = [int][System.Convert]::ToBoolean($ConfigFile.RemotePrograms.DeploymentSettings.GatewaySettings.UseCachedCreds)
 	$values["allow font smoothing:i"] = [int][System.Convert]::ToBoolean($ConfigFile.RemotePrograms.DeploymentSettings.CertificateSettings.AllowFontSmoothing)
 	$values["session bpp:i"] = $ConfigFile.RemotePrograms.DeploymentSettings.CertificateSettings.ColorBitDepth
 	$values["use multimon:i"] = [int][System.Convert]::ToBoolean($ConfigFile.RemotePrograms.DeploymentSettings.CertificateSettings.UseMultimon)
 	$values["authentication level:i"] = $ConfigFile.RemotePrograms.DeploymentSettings.ServerAuth
 	
 	switch ($ConfigFile.RemotePrograms.DeploymentSettings.GatewaySettings.GatewayUsage)	{
 		0 {$values["gatewayusagemethod:i"] = 0;$values["gatewayprofileusagemethod:i"] = 1}
 		1 {$values["gatewayusagemethod:i"] = 2;$values["gatewayprofileusagemethod:i"] = 1}
 		2 {$values["gatewayusagemethod:i"] = 1;$values["gatewayprofileusagemethod:i"] = 1}
 		3 {$values["gatewayusagemethod:i"] = 2;$values["gatewayprofileusagemethod:i"] = 0}
 		default	{$values["gatewayusagemethod:i"] = 0;$values["gatewayprofileusagemethod:i"] = 1}
 	}
 
 	switch ($ConfigFile.RemotePrograms.DeploymentSettings.CertificateSettings.RedirectionSettings)	{
 		{$_ -band 1} {$values["redirectdrives:i"] = 1;$values["drivestoredirect:s"] = "*"}
 		{$_ -band 2} {$values["redirectprinters:i"] = 1}
 		{$_ -band 4} {$values["redirectclipboard:i"] = 1}
 		{$_ -band 8} {$values["devicestoredirect:s"] = "*"}
 		{$_ -band 16} {$values["redirectsmartcards:i"] = 1}
 	}
 
 	$DeploymentSettings.DeploymentRDPSettings = [string]::Join("`n",($values.Keys | ForEach-Object {$_ + ":" + $values[$_]}))
 	$DeploymentSettings.CustomRDPSettings = [System.Convert]::ToString($ConfigFile.RemotePrograms.DeploymentSettings.CertificateSettings.CustomRDPSettings)
 	$DeploymentSettings.CertificateExpiresOn = [int64]$ConfigFile.RemotePrograms.DeploymentSettings.CertificateSettings.CertificateExpiresOn
 	$DeploymentSettings.CertificateIssuedTo = [System.Convert]::ToString($ConfigFile.RemotePrograms.DeploymentSettings.CertificateSettings.CertificateIssuedTo)
 	$DeploymentSettings.CertificateIssuedBy = [System.Convert]::ToString($ConfigFile.RemotePrograms.DeploymentSettings.CertificateSettings.CertificateIssuedBy)
 	$DeploymentSettings.HasCertificate = [System.Convert]::ToBoolean($ConfigFile.RemotePrograms.DeploymentSettings.CertificateSettings.UseCertificate)
 	if ($ConfigFile.RemotePrograms.DeploymentSettings.CertificateSettings.Certificate){$DeploymentSettings.CertificateHash = @($ConfigFile.RemotePrograms.DeploymentSettings.CertificateSettings.Certificate -split '([a-f0-9]{2})' | foreach-object { if ($_) {[System.Convert]::ToByte($_,16)}})}
 	$DeploymentSettings.Put() | Out-Null
 	if ($?) {Write-Host "The settings was succesfully entered" -ForegroundColor yellow}
 
 	if ($ConfigFile.RemotePrograms.DeploymentSettings.CertificateSettings.UseCertificate.ShowRemoteDesktop) {
 		$newapp = ([wmiclass]"ROOT\CIMV2\TerminalServices:Win32_TSRemoteDesktop").CreateInstance()
 		$newapp.Alias = "TSRemoteDesktop"
 		$newapp.IconIndex = 0
 		$newapp.Name = "Remote Desktop"
 		$newapp.ShowInPortal = $true
 		$newapp.RDPFileContents=(Get-WmiObject -Class "Win32_TSDeploymentSettings" -Namespace "root\CIMV2\TerminalServices").DeploymentRDPSettings
 		$newapp.iconpath = "%SYSTEMDRIVE%\Windows\system32\mstsc.exe"
 		$newapp.Put() | Out-Null
 	} else {Get-WmiObject -Class "Win32_TSRemoteDesktop" -Namespace "root\CIMV2\TerminalServices" | ForEach-Object {$_.Delete() | Out-Null}}
 
 }
 
 foreach ($Application in $ConfigFile.RemotePrograms.Application) {
 	if ((Get-WmiObject -Namespace "root\CIMV2\TerminalServices" -Query "select * from Win32_TSPublishedApplication where alias = '$Application.Alias'") -ne $null) {
 		Write-Warning "The application $Application.Alias already exist!"
 		continue
 	}
 	$newapp = ([wmiclass]"ROOT\CIMV2\TerminalServices:Win32_TSPublishedApplication").CreateInstance()
 	$newapp.Alias = $Application.Alias
 	$newapp.Path = $Application.Path
 	$newapp.VPath = $Application.VPath
 	$newapp.ShowInPortal = ($Application.ShowInTSWA -eq "Yes")
 	$newapp.Name = $Application.Name
 	$newapp.CommandLineSetting = $Application.CommandLineSetting
 	$newapp.RequiredCommandLine = $Application.RequiredCommandLine
 	$newapp.IconIndex = $Application.IconIndex
 	$newapp.IconPath = $Application.IconPath
 	$newapp.RDPFileContents = $Application.RDPContents
 	if ((Get-WmiObject -Class Win32_OperatingSystem).Version -ge 6.1){$newapp.SecurityDescriptor = $Application.SecurityDescriptor}
 	$newapp.Put() | Out-Null
 	if ($?) {Write-Host "The application $($Application.Alias) was succesfully created" -ForegroundColor yellow}
 }
 
 }
 
 function Export-TSRemoteAppsSet {
 <#
 .SYNOPSIS
 Exports all TS RemoteApps from the Windows 2008 Terminal Server.
 .DESCRIPTION
 Exports all TS RemoteApps from the Windows 2008 Terminal Server to a TSPUB-file.
 One mandatory parameter: Path
 .PARAMETER path
 Path to the TSPUB-file to be exported
 .EXAMPLE
 Export-TSRemoteApps -Path C:\temp\RemoteApps.tspub
 Exports all RemoteApps to the specified TSPUB-file.
 .NOTES
 AUTHOR:    Wizarden
 LASTEDIT:  22.11.2010 
 #>
 
 [CmdletBinding()]
     param (
         [parameter(Mandatory=$true,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
         [string]$Path
     )
 
 $xmldata = New-Object xml
 $newelement = $xmldata.CreateXmlDeclaration("1.0", $null, $null)
 $xmldata.AppendChild($newelement) | Out-Null
 $newelement = $xmldata.CreateElement("RemotePrograms")
 
 $newelement.InnerXml = "<UseAllowList />"
 
 $Win32_TSPublishedApplicationList = Get-WmiObject -Namespace "root\CIMV2\TerminalServices" -Class Win32_TSPublishedApplicationList
 if ($Win32_TSPublishedApplicationList.Disabled){$newelement.UseAllowList = "No"} else {$newelement.UseAllowList = "Yes"}
 
 $xmldata.AppendChild($newelement) | Out-Null
 
 $DeploymentSettingsElement = $xmldata.CreateElement("DeploymentSettings")
 $GatewaySettingsElement = $xmldata.CreateElement("GatewaySettings")
 $CertificateSettingsElement = $xmldata.CreateElement("CertificateSettings")
 #$newelement.InnerXml = "<UseAllowList /><DeploymentSettings><Port /><FarmName /><GatewaySettings><GatewayName /><GatewayUsage /><GatewayAuthMode />
 #<UseCachedCreds /></GatewaySettings><CertificateSettings><UseCertificate /><CertificateSize /><Certificate /><CertificateIssuedTo />
 #<CertificateIssuedBy /><CertificateExpiresOn /><AllowFontSmoothing /><UseMultimon /><ColorBitDepth /><RedirectionSettings /><CustomRdpSettings />
 #<ShowRemoteDesktop /><RemoteDesktopSecurityDescriptor /><UseMultimon /></CertificateSettings></DeploymentSettings>"
 $DeploymentSettings = Get-WmiObject -Namespace "root\CIMV2\TerminalServices" -Class Win32_TSDeploymentSettings
 
 if ($DeploymentSettings.Port){$DeploymentSettingsElement.InnerXml += "<Port />";$DeploymentSettingsElement.Port = $DeploymentSettings.Port.ToString()}
 $DeploymentSettingsElement.InnerXml += "<ServerAuth />";$DeploymentSettingsElement.ServerAuth = [string][int[]]($DeploymentSettings.RequireServerAuth)
 if ($DeploymentSettings.FarmName){$DeploymentSettingsElement.InnerXml += "<FarmName />";$DeploymentSettingsElement.FarmName = $DeploymentSettings.FarmName.ToString()}
 
 if ($DeploymentSettings.GatewayName){$GatewaySettingsElement.InnerXml += "<GatewayName />";$GatewaySettingsElement.GatewayName = $DeploymentSettings.GatewayName.ToString()}
 $GatewaySettingsElement.InnerXml += "<GatewayUsage />";$GatewaySettingsElement.GatewayUsage = $DeploymentSettings.GatewayUsage.ToString()
 $GatewaySettingsElement.InnerXml += "<GatewayAuthMode />";$GatewaySettingsElement.GatewayAuthMode = $DeploymentSettings.GatewayAuthMode.ToString()
 $GatewaySettingsElement.InnerXml += "<UseCachedCreds />";$GatewaySettingsElement.UseCachedCreds = $DeploymentSettings.GatewayUseCachedCreds.ToString()
 
 $CertificateSettingsElement.InnerXml += "<UseCertificate />";$CertificateSettingsElement.UseCertificate = $DeploymentSettings.HasCertificate.ToString()
 if ($DeploymentSettings.CertificateHash){
 	$CertificateSettingsElement.InnerXml += "<CertificateSize /><Certificate />"
 	$CertificateSettingsElement.CertificateSize = $DeploymentSettings.CertificateHash.Length.ToString()
 	$CertificateSettingsElement.Certificate = ([System.BitConverter]::ToString($DeploymentSettings.CertificateHash)).Replace("-","")
 }
 if ($DeploymentSettings.CertificateIssuedTo){$CertificateSettingsElement.InnerXml += "<CertificateIssuedTo />";$CertificateSettingsElement.CertificateIssuedTo = $DeploymentSettings.CertificateIssuedTo.ToString()}
 if ($DeploymentSettings.CertificateIssuedBy){$CertificateSettingsElement.InnerXml += "<CertificateIssuedBy />";$CertificateSettingsElement.CertificateIssuedBy = $DeploymentSettings.CertificateIssuedBy.ToString()}
 if ([int64]$DeploymentSettings.CertificateExpiresOn){$CertificateSettingsElement.InnerXml += "<CertificateExpiresOn />";$CertificateSettingsElement.CertificateExpiresOn = $DeploymentSettings.CertificateExpiresOn.ToString()}
 $CertificateSettingsElement.InnerXml += "<AllowFontSmoothing />";$CertificateSettingsElement.AllowFontSmoothing = $DeploymentSettings.AllowFontSmoothing.ToString()
 if ($DeploymentSettings.UseMultimon -ne $null){$CertificateSettingsElement.InnerXml += "<UseMultimon />";$CertificateSettingsElement.UseMultimon = $DeploymentSettings.UseMultimon.ToString()}
 $CertificateSettingsElement.InnerXml += "<ColorBitDepth />";$CertificateSettingsElement.ColorBitDepth = $DeploymentSettings.ColorBitDepth.ToString()
 $CertificateSettingsElement.InnerXml += "<RedirectionSettings />";$CertificateSettingsElement.RedirectionSettings = $DeploymentSettings.RedirectionOptions.ToString()
 if ($DeploymentSettings.CustomRDPSettings){$CertificateSettingsElement.InnerXml += "<CustomRdpSettings />";$CertificateSettingsElement.CustomRdpSettings = $DeploymentSettings.CustomRDPSettings.ToString()}
 $CertificateSettingsElement.InnerXml += "<ShowRemoteDesktop />";$CertificateSettingsElement.ShowRemoteDesktop = [string][bool](Get-WmiObject -Namespace "root\CIMV2\TerminalServices" -Class Win32_TSRemoteDesktop)
 $CertificateSettingsElement.InnerXml += "<RemoteDesktopSecurityDescriptor />"
 
 if ($GatewaySettingsElement.InnerXml -ne ""){$DeploymentSettingsElement.AppendChild($GatewaySettingsElement) | Out-Null}
 if ($CertificateSettingsElement.InnerXml -ne ""){$DeploymentSettingsElement.AppendChild($CertificateSettingsElement) | Out-Null}
 if ($DeploymentSettingsElement.InnerXml -ne ""){$xmldata.RemotePrograms.AppendChild($DeploymentSettingsElement) | Out-Null}
 Write-Host "The settings was succesfully exported" -ForegroundColor yellow
 
 Get-WmiObject -Namespace "root\CIMV2\TerminalServices" -Class Win32_TSPublishedApplication | ForEach-Object {
 	$newelement = $xmldata.CreateElement("Application")
 
 	if ($_.Name){$newelement.InnerXml += "<Name />";$newelement.Name = $_.Name.ToString()}
 	if ($_.Alias){$newelement.InnerXml += "<Alias />";$newelement.Alias = $_.Alias.ToString()}
 	$newelement.InnerXml += "<SecurityDescriptor />";if ($_.SecurityDescriptor){$newelement.SecurityDescriptor = $_.SecurityDescriptor.ToString()}
 	if ($_.Path){$newelement.InnerXml += "<Path />";$newelement.Path = $_.Path.ToString()}
 	$newelement.InnerXml += "<VPath />";if ($_.VPath){$newelement.VPath = $_.VPath.ToString()}
 	if ($_.ShowInPortal){$newelement.InnerXml += "<ShowInTSWA />";$newelement.ShowInTSWA = "Yes"} else {$newelement.InnerXml += "<ShowInTSWA />";$newelement.ShowInTSWA = "No"}
 	$newelement.InnerXml += "<RequiredCommandLine />";if ($_.RequiredCommandLine){$newelement.RequiredCommandLine = $_.RequiredCommandLine.ToString()}
 	if ($_.IconPath){$newelement.InnerXml += "<IconPath />";$newelement.IconPath = $_.IconPath.ToString()}
 	$newelement.InnerXml += "<IconIndex />";$newelement.IconIndex = $_.IconIndex.ToString()
 	$newelement.InnerXml += "<CommandLineSetting />";$newelement.CommandLineSetting = $_.CommandLineSetting.ToString()
 	if ($_.RDPFileContents){$newelement.InnerXml += "<RDPContents />";$newelement.RDPContents = $_.RDPFileContents.ToString()}
 
 	$xmldata.RemotePrograms.AppendChild($newelement) | Out-Null
 	Write-Host "The $($newelement.Alias) was succesfully exported" -ForegroundColor yellow
 }
 
 $xmldata.Save($Path)
 if ($?) {Write-Host "The $Path saved was succesfully" -ForegroundColor yellow}
 
 }
`

