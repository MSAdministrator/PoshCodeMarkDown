---
Author: zefram
Publisher: 
Copyright: 
Email: 
Version: 4.0
Encoding: ascii
License: cc0
PoshCode ID: 5620
Published Date: 2014-12-01t16
Archived Date: 2014-12-14t00
---

# order importer - 

## Description

etl from shopping cart to mom

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 ###
 #
 ###
 
 ################
 ################
 
 [CmdletBinding()]
 param(
 )
 
 #####################
 #####################
 
 $error.clear()
 
 $ErrorActionPreference = "Stop"
 
 if((Get-Variable MOM_API -ErrorAction SilentlyContinue) -ne $null)
 {
 	Remove-Variable MOM_API
 }
 
 ###############
 ###############
 
 [string]$ScriptName = $MyInvocation.MyCommand.Name
 
 [string]$ScriptPath = '\\NTSERVER1\Batch\online_orders_test'
 
 [string]$global:logFile = '\\NTSERVER1\Logs\internet_orders_processing.log'
 
 [string]$global:syslog = '\\NTSERVER1\Automation\scripts\Dependencies\New-SyslogSender.ps1'
 
 [string]$ValidateAddressScript = '\\NTSERVER1\Automation\scripts\UPSValidate-Address.ps1'
 [string]$ValidateXMLScript = '\\NTSERVER1\Automation\scripts\Dependencies\MomImporting\Validate-Xml.ps1'
 
 [string]$ArchiveDir = "\\NTSERVER1\Data\temp\xml_archive\"
 
 $XMLDownloadsPath = "downloaded_orders"
 $XMLErroroneousOrdersPath = "$ScriptPath\$XMLDownloadsPath\Erroneous_Orders"
 $MOMXMLOutputPath = "AOG_to_XML_output"
 $AOGCSVOutputPath = "AOG_CSV_output"
 
 [string]$ftpServer = 'ftp://nametag.com'
 [string]$FTPLocalOrdersDir = "$ScriptPath\$XMLDownloadsPath\"
 [string]$FTPLocalListsDir = '\\NTSERVER1\Data\temp\lists_download\'
 
 [string]$ServerInstance = 'NTSERVER1'
 $ConnectionTimeout = 5
 $QueryTimeout = 10
 
 [string]$MOMUser = 'NET'
 [string]$MOMPass = ''
 
 [string]$AOGEXEPath = '\\NTSERVER1\NT_Programs\AOG\ordergen.exe'
 
 [string]$MOMImportSchemaPath = "\\NTSERVER1\Automation\scripts\Dependencies\MomImporting\momxmltemplate.xsd"
 [string]$MOMImportXMLTemplate = "\\NTSERVER1\Automation\scripts\Dependencies\MomImporting\momxmltemplate.xml"
 [string]$MOMXMLOutputFile = "$ScriptPath\$MOMXMLOutputPath\orderimport.xml"
 [string]$CartXMLSchemaPath = "\\NTSERVER1\Automation\scripts\Dependencies\MomImporting\cartxmltemplate.xsd"
 
 [string]$CurrentCustomersPath = '\\NTSERVER1\Data\Customers'
 [string]$OldCustomersPath = "$CurrentCustomersPath\_Old"
 
 ###############
 ###############
 
 Function exception_handler 
 {
 	param 
 	(
 		[parameter(Mandatory=$true)]
 		[string]$Message,
 		[parameter(Mandatory=$true)]
 		[string]$Severity,
 		[parameter(Mandatory=$false)]
 		[string]$AttachmentPath,
 		[parameter(Mandatory=$false)]
 		[string]$MoveFile,
 		[parameter(Mandatory=$false)]
 		[switch]$NoEmail
 	)
 
 	if(-Not ($SyslogObj = Invoke-Expression $global:syslog))
 	{
 		Write-Error "Could not load syslog script at path: $global:syslog"
 		Exit
 	}
 		
 	if((Get-Variable MOM_API -ErrorAction SilentlyContinue) -ne $null)
 	{
 		if($MOM_API.Error_Msg() -ne '')
 		{
 			$Message =	$Message = "MOM API Error: $($MOM_API.Error_Msg()) `n`n"
 		}
 	}
 	
 	$t = Get-Date -Format "M-dd H:mm"
 	$SyslogObj.Send("$Message", 1, $Severity, $null, "$t - Online Importer Test - ")
 	
 	Add-Content $global:logFile "$t - $Message " -ErrorAction Continue
 	
 	if(!$NoEmail)
 	{
 		$Message = $Message.Replace('"',"'")
 		
 		$EmailExp = "Send-MailMessage -SmtpServer 'zimbra.xmission.com' -To 'jeffrey@nametag.com' -Cc 'rod@nametag.com','clyde@nametag.com','joshua@nametag.com' -From 'nametag@nametag.com' -Subject 'TEST - Automatic Order Import Error' -Body `"$Message`""
 		if($AttachmentPath -ne '')
 		{
 			$EmailExp += " -Attachments $AttachmentPath"
 		}
 		
 		Write-Debug $EmailExp
 		Invoke-Expression -Command $EmailExp
 	}
 	
 	if($MoveFile -and (Test-Path $MoveFile) -and -not $NoDeleteLocal)
 	{
 		Move-Item $MoveFile $XMLErroroneousOrdersPath -Force
 	}
 	
 	Return $Message
 }
 
 Function load_snapin
 {
 	param (
 	[parameter(Mandatory=$true)]
 	[string]$SnapinName 
 	)
 
 	if ((Get-PSSnapin $SnapinName -Registered -ErrorAction SilentlyContinue) -eq $null)
 	{
 		Write-Debug "$SnapinName is not loaded. Attempting to registered and load snapin..."
 		try
 		{
 			set-alias installutil $env:windir\microsoft.net\framework\v2.0.50727\installutil
 			installutil -i "C:\Program Files (x86)\Microsoft SQL Server\100\Tools\Binn\Microsoft.SqlServer.Management.PSProvider.dll"
 			installutil -i "C:\Program Files (x86)\Microsoft SQL Server\100\Tools\Binn\Microsoft.SqlServer.Management.PSSnapins.dll"
 		}
 		catch [System.Exception]
 		{
 			exception_handler -Message "Unable to register $SnapinName. Is SQL Server installed?: $_" -Severity 'critical'
 		}
 	}
 				
 	if ((Get-PSSnapin -Name $SnapinName -ErrorAction SilentlyContinue) -eq $null )
 	{
 		try 
 		{
 			Add-PSSnapin $SnapinName 
 		}
 		catch [System.Exception]
 		{
 			exception_handler -Message "Unable to load ${$SnapinName}: $_" -Severity 'critical'
 		}
 	}
 }
 
 Function insert_new_customer
 {
 	$Result = Invoke-Sqlcmd -Query "[dbo].[create_new_customer]" -ServerInstance $ServerInstance -Database $Database -ConnectionTimeout $ConnectionTimeout -QueryTimeout $QueryTimeout | Select-Object -ExpandProperty Column1
 	Write-Debug "Custnum created in the database: $Result"
 	return $Result
 }
 
 . $ValidateXMLScript
 
 . $ValidateAddressScript
 
 #################
 #################
 
 if(-Not ($PSVersionTable.PSVersion.Major -ge 4))
 {
 	Throw exception_handler -Message "This script must be run in PowerShell v4.0 or greater" -Severity 'critical'
 }
 
 if(-Not (Get-Module -name "PSFTP")) {
 	try
 	{
 		Write-Verbose ("Loading PSFTP module.")
 		Import-Module PSFTP
 	}
 	catch [System.Exception]
 	{
 		exception_handler -Message "Error loading PSFTP module: $_" -Severity 'critical'
 	}
 }
 
 try 
 {
 	load_snapin -SnapinName "SqlServerCmdletSnapin100"
 	load_snapin -SnapinName "SqlServerProviderSnapin100"	
 }
 catch [System.Exception]
 {
 	exception_handler -Message "Error loading SqlServer snapins`: $_" -Severity 'critical'
 }
 
 $folders = @($XMLDownloadsPath,$MOMXMLOutputPath,$AOGCSVOutputPath)
 ForEach ($folder in $folders)
 {
 	try 
 	{
 		if(-Not (Test-Path "$ScriptPath\$folder"))
 		{
 			New-Item "$ScriptPath\$folder" -Type Directory | Out-Null
 		}
 	}
 	catch [System.Exception]
 	{
 		exception_handler -Message "Error creating missing folder `"$folder`"`: $_" -Severity 'critical'
 	}
 }
 
 if((Import-Module '\\NTSERVER1\Automation\scripts\Dependencies\MomImporting\MomApiWrapper.dll' -ErrorAction SilentlyContinue) -ne $null)
 {
 	Throw exception_handler -Message "Unable to load MOM API wrapper DLL." -Severity 'critical'
 }
 
 if(-Not (Test-Path 'C:\MOMLocal*\MOM SDK\MOMAPI*'))
 {
 	Throw exception_handler -Message "MOM API does not appear to be installed." -Severity 'critical'
 }
 
 try
 {
 	if($MOM_API -eq $null)
 	{
 		$MOM_API = New-Object MomApiWrapper.MomApiWrapper
 		
 		if($MOM_API.DBConnect("Driver=SQL Server;server=$ServerInstance;database=$DatabaseName;trusted_connection=yes") -ne 1)
 		{
 			Throw "MOM API could not connect to database.`nServer: $ServerInstance`nDatabase=$DatabaseName"
 		}
 		else
 		{
 			Write-Host "MOM API connected to database successfully."
 		}
 		
 		if($MOM_API.MOMLogin("$MOMUser","$MOMPass") -ne 1)
 		{
 			Throw "MOM API could not login.`n User:$MOMUser`n Pass:$MOMPass"
 		}
 		else
 		{
 			Write-Host "MOM API logged in successfully."
 		}
 		
 		if($MOM_API.Error_Msg() -ne '')
 		{
 			Throw
 		}
 	}
 }
 catch [System.Exception]
 {
 	exception_handler -Message "Unable to import MomApiWrapper.dll and connect to DB: $_" -Severity 'error'
 }
 
 ####################
 ####################
 if(!$NoDownload)
 {
 	try
 	{
 		Write-Verbose ("Gathering credentials.")
 		$pass = Get-Content $ScriptPath\pass.txt | ConvertTo-SecureString
 	}
 	catch [System.Exception]
 	{
 		exception_handler -Message "Error retrieving password file`: $_" -Severity 'critical'
 	}
 
 	try
 	{
 		Write-Verbose ("Building credentials object.")
 		$creds = New-Object System.Management.Automation.PsCredential('nametag', $pass)
 	}
 	catch [System.Exception]
 	{
 		exception_handler -Message "$_" -Severity 'critical'
 	}
 	
 	$i = 1
 	while($i -le 3)
 	{
 		try
 		{
 			Write-Verbose ("Building FTP connection.")
 			Set-FTPConnection -Credentials $creds -Server $ftpServer -Session ftpSessionInit -UsePassive -UseBinary | Out-Null
 		}
 		catch [System.Exception]
 		{
 			exception_handler -Message "Error setting up FTP connection: $_" -Severity 'critical' -NoEmail
 			$i++
 			Start-Sleep 10
 		}
 
 		try
 		{
 			Write-Verbose ("Starting FTP session.")
 			$ftpSession = Get-FTPConnection -Session ftpSessionInit
 		}
 		catch [System.Exception]
 		{
 			exception_handler -Message "Error connecting to FTP Server: $_" -Severity 'critical' -NoEmail
 			$i++
 			Start-Sleep 10
 		}
 
 		#####################
 		#####################
 		
 		try 
 		{
 			Write-Verbose ("Gathering remote orders.")
 			
 			$remoteFiles = Get-FTPChildItem -Session $ftpSession -Path $FTPRemoteOrdersDir -Filter $includeExtOrders | Where-Object {$_.Size -ne ""} | Select-Object -Property ModifiedDate, Name
 			
 			$filesToDownload = $remoteFiles | Where-Object {$_.InputObject -ne ""}
 			
 			Write-Verbose ("Orders to download:`n" + $filesToDownload.Name)
 		} 
 		catch [System.Exception] 
 		{
 			if($i -eq 3)
 			{
 				exception_handler -Message "Error retrieving remote orders: $_" -Severity 'critical'
 			}
 			else
 			{
 				$i++
 				Start-Sleep 10
 			}	
 		}
 	}
 	
 	Write-Debug("Remote orders:`n$remoteFiles")
 
 	foreach($file in $filesToDownload.Name) 
 	{
 		$filepath = $FTPRemoteOrdersDir+$file
 		Write-Verbose("Current file path:$filepath`n")
 		
 		try 
 		{
 			Get-FTPItem -Path $filepath -LocalPath $FTPLocalOrdersDir -Session $ftpSession -Overwrite | Out-Null
 			Write-Verbose ("Downloaded: $file")
 		}
 		catch [System.Exception] 
 		{
 			exception_handler -Message "Error downloading orders: $_" -Severity 'critical'
 		}
 		
 		if(!$NoDeleteRemote -and (Test-Path ($FTPLocalOrdersDir+$file)))
 		{
 			try 
 			{		
 				Remove-FTPItem -Path $filepath -Session $ftpSession
 				Write-Verbose ("Deleted: $file")
 			} 
 			catch [System.Exception] 
 			{
 				exception_handler -Message "Error deleting files: $_" -Severity 'critical'
 			}
 		}
 	}
 
 	####################
 	####################
 
 	$i = 1
 	while($i -le 3)
 	{
 		try 
 		{
 			Write-Verbose ("Gathering remote orders.")
 			
 			$remoteFiles = Get-FTPChildItem -Session $ftpSession -Path $FTPRemoteListsDir -Filter $includeExtLists | Where-Object {$_.Size -ne ""} | Select-Object -Property ModifiedDate, Name
 			$remoteFiles | ForEach-Object { $_.Name = $_.Name.Trim() }
 			
 			$filesToDownload = $remoteFiles | Where-Object {$_.InputObject -ne "" }
 			
 			Write-Verbose ("Lists to download:`n" + $filesToDownload.Name)
 			Break
 		} 
 		catch [System.Exception] 
 		{
 			if($i -eq 3)
 			{
 				exception_handler -Message "Error retrieving remote lists: $_" -Severity 'critical'
 			}
 			else
 			{
 				$i++
 				Start-Sleep 10
 			}	
 		}
 	}
 	Write-Debug("Remote lists:`n$remoteFiles")
 	
 	$CharsString = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz123456789 !@#$%^&*()[]-=_+/\{}|:;''",.<>?`~'
 	$CharsString = [regex]::escape($CharsString)
 	
 	foreach($file in $filesToDownload.Name) 
 	{
 		$filepath = $FTPRemoteListsDir+$file
 		Write-Verbose("Current file path:$filepath`n")
 		
 		try 
 		{
 			Get-FTPItem -Path $filepath -LocalPath $FTPLocalListsDir -Session $ftpSession -Overwrite | Out-Null
 			Write-Verbose ("Downloaded: $file")
 		} 
 		catch [System.Exception] 
 		{
 			exception_handler -Message "Error downloading lists: $_`n`nFile`: $file" -Severity 'critical'
 		}
 		
 		if(Test-Path ($FTPLocalListsDir+$file))
 		{
 			$ListString = Get-Content $($FTPLocalListsDir+$file) | Out-String
 
 			$ListString = $ListString -replace("`t|`r|`n","")
 
 			$InvalidCharsFound = @()
 			foreach($c in $ListString.GetEnumerator())
 			{
 				if(-not [regex]::Match("$c","[$CharsString]").Success)
 				{
 					$InvalidCharsFound += $c
 				}
 			}
 		}
 		
 		if(!$NoDeleteRemote -and (Test-Path ($FTPLocalListsDir+$file)))
 		{
 			try 
 			{		
 				Remove-FTPItem -Path $filepath -Session $ftpSession
 				Write-Verbose ("Deleted: $file")
 			} 
 			catch [System.Exception] 
 			{
 				exception_handler -Message "Error deleting files: $_`n`nFile`: $file" -Severity 'critical'
 			}
 		}
 	}
 	
 }
 ################################
 ################################
 
 $Orders = Get-ChildItem -Path "$FTPLocalOrdersDir" | Where-Object {$_.PSIsContainer -eq $False} | Sort-Object -Property LastWriteTime -Descending
 
 if($(Get-Process -ProcessName "*ordergen*") -ne $null)
 {
 	Write-Verbose "AOG is running! Exiting."
 	Throw exception_handler -Message "AOG is running! AOG failed to exit itself, or it's open by a user. Killing AOG for now. This should be fixed." -Severity 'warning'
 	
 	Stop-Process -ProcessName "*ordergen*"
 }
 
 if($Orders -eq $null)
 {
 	Write-Verbose "No orders to import."
 	Exit
 }
 
 foreach ($Order in $Orders) 
 {
 	#######################
 	#######################
 
 	try
 	{
 		Remove-Item -Force "$ScriptPath\AOG_to_XML_output\*"
 		Remove-Item -Force "$ScriptPath\AOG_CSV_output\*"
 
 		Copy-Item $Order.FullName $ArchiveDir$Order
 	}
 	catch [System.Exception]
 	{
 		exception_handler -Message "$($Order.Name) - Unable to load XML file at path $ShoppingCartXMLPath`n`n $_" -Severity 'error'
 		continue
 	}
 	
 	try
 	{
 		Validate-XML -XmlFile $Order.FullName -SchemaFile $CartXMLSchemaPath
 	}
 	catch [System.Exception]
 	{
 		exception_handler -Message "$($Order.Name) - XML file is invalid`n`n $_" -Severity 'error' -MoveFile $Order.FullName
 		continue
 	}	
 	
 	try
 	{
 		$OrderXML = New-Object -TypeName XML
 		$OrderXML.PreserveWhitespace = $True
 		$OrderXML.Load($Order.FullName)
 		
 		$InnerXML = Select-Xml -Xml $OrderXML -XPath '//VFPData'
 		
 		
 		$AltNum = $InnerXML.Node.tagorderxml.ordernum
 		if($AltNum -eq $null)
 		{
 			Throw 'No AltNum in XML for order.'
 			continue
 		}
 	}
 	catch [System.Exception]
 	{
 		exception_handler -Message "$($Order.Name) - Unable to load XML file at path $ShoppingCartXMLPath`n`n $_" -Severity 'error' -MoveFile $Order.FullName
 		continue
 	}
 
 	#######################
 	#######################
 	
 	if($InnerXML.Node.tagorderxml.bcustnum -eq '' -and $InnerXML.Node.tagorderxml.baddress1 -ne '') 
 	{
 		Write-Verbose '<bcustnum> is empty. Adding billing customer to MOM.'
 		
 		try
 		{
 			$NewBillCust = insert_new_customer
 			
 			$InnerXML.Node.tagorderxml.bcustnum = "$NewBillCust"	
 			
 			$Query = "
 			UPDATE CUST SET
 			FIRSTNAME = '$($InnerXML.Node.tagorderxml.bfirstname)',
 			LASTNAME = '$($InnerXML.Node.tagorderxml.blastname)',
 			COMPANY = '$($InnerXML.Node.tagorderxml.bcompany)',
 			ADDR = '$($InnerXML.Node.tagorderxml.baddress1)',
 			ADDR2 = '$($InnerXML.Node.tagorderxml.baddress2)',
 			CITY = '$($InnerXML.Node.tagorderxml.bcity)',
 			STATE = '$($InnerXML.Node.tagorderxml.bstate)',
 			ZIPCODE = '$($InnerXML.Node.tagorderxml.bzip)',
 			COUNTRY = '$($InnerXML.Node.tagorderxml.bcountrycode)',
 			PHONE = '$($InnerXML.Node.tagorderxml.bphone1)',
 			EXTENSION = '$($InnerXML.Node.tagorderxml.bphext1)',
 			EMAIL = '$($InnerXML.Node.tagorderxml.bemail)',
 			UPSCOMDELV = '$($InnerXML.Node.tagorderxml.baddrtype)'
 			WHERE custnum = '$NewBillCust'"
 			$Result = Invoke-Sqlcmd -Query $Query -ServerInstance $ServerInstance -Database $Database -ConnectionTimeout $ConnectionTimeout -QueryTimeout $QueryTimeout
 		
 		}
 		catch [System.Exception]
 		{
 			exception_handler -Message "$($Order.Name) - Error creating new billing customer: $_" -Severity 'error' -MoveFile $Order.FullName
 			continue
 		}
 		
 		$InnerXML.Node.tagorderxml.bcustnum = "$NewBillCust"
 	}
 	
 	if($InnerXML.Node.tagorderxml.scustnum -eq '' -and $InnerXML.Node.tagorderxml.saddress1 -ne '')
 	{
 		Write-Verbose '<scustnum> is empty. Adding customer to MOM.'
 		
 		try
 		{
 			$NewShipCust = insert_new_customer
 			
 			$InnerXML.Node.tagorderxml.scustnum = "$NewShipCust"	
 				
 			$Query = "
 			UPDATE CUST SET
 			FIRSTNAME = '$($InnerXML.Node.tagorderxml.sfirstname)',
 			LASTNAME = '$($InnerXML.Node.tagorderxml.slastname)',
 			COMPANY = '$($InnerXML.Node.tagorderxml.scompany)',
 			ADDR = '$($InnerXML.Node.tagorderxml.saddress1)',
 			ADDR2 = '$($InnerXML.Node.tagorderxml.saddress2)',
 			CITY = '$($InnerXML.Node.tagorderxml.scity)',
 			STATE = '$($InnerXML.Node.tagorderxml.sstate)',
 			ZIPCODE = '$($InnerXML.Node.tagorderxml.szip)',
 			COUNTRY = '$($InnerXML.Node.tagorderxml.scountrycode)',
 			PHONE = '$($InnerXML.Node.tagorderxml.sphone1)',
 			EXTENSION = '$($InnerXML.Node.tagorderxml.sphext1)',
 			EMAIL = '$($InnerXML.Node.tagorderxml.semail)',
 			UPSCOMDELV = '$($InnerXML.Node.tagorderxml.saddrtype)'
 			WHERE custnum = '$NewShipCust'"
 			$Result = Invoke-Sqlcmd -Query $Query -ServerInstance $ServerInstance -Database $Database -ConnectionTimeout $ConnectionTimeout -QueryTimeout $QueryTimeout	
 			
 			$Query = "
 			INSERT INTO CUSTRELA (BELONGNUM,RELA_TYPE,CUSTNUM,DESC1) VALUES ($NewShipCust,'S',$($InnerXML.Node.tagorderxml.scustnum),'')
 			INSERT INTO CUSTRELA (BELONGNUM,RELA_TYPE,CUSTNUM,DESC1) VALUES ($($InnerXML.Node.tagorderxml.scustnum),'B',$NewShipCust,'')"
 			$Result = Invoke-Sqlcmd -Query $Query -ServerInstance $ServerInstance -Database $Database -ConnectionTimeout $ConnectionTimeout -QueryTimeout $QueryTimeout
 		}
 		catch [System.Exception]
 		{
 			exception_handler -Message "$($Order.Name) - Error creating new shipping customer: $_" -Severity 'error' -MoveFile $Order.FullName
 			continue
 		}		
 	}
 
 	if($InnerXML.Node.tagorderxml.ccustnum -eq '' -and ($InnerXML.Node.tagorderxml.cfirstname -ne '' -or $InnerXML.Node.tagorderxml.clastname -ne '') -and $InnerXML.Node.tagorderxml.xml_type -eq 'OLR')
 	{
 		Write-Verbose '<ccustnum> is empty and <xml_type> is OLR. Adding customer to MOM.'
 		
 		try
 		{
 			$NewSoldToCust = insert_new_customer
 			
 			$InnerXML.Node.tagorderxml.ccustnum = "$NewSoldToCust"	
 			
 			$Query = "
 			UPDATE CUST SET
 			FIRSTNAME = '$($InnerXML.Node.tagorderxml.cfirstname)',
 			LASTNAME = '$($InnerXML.Node.tagorderxml.clastname)',
 			COMPANY = '$($InnerXML.Node.tagorderxml.ccompany)',
 			ADDR = '$($InnerXML.Node.tagorderxml.caddress1)',
 			ADDR2 = '$($InnerXML.Node.tagorderxml.caddress2)',
 			CITY = '$($InnerXML.Node.tagorderxml.ccity)',
 			STATE = '$($InnerXML.Node.tagorderxml.cstate)',
 			ZIPCODE = '$($InnerXML.Node.tagorderxml.czip)',
 			COUNTRY = '$($InnerXML.Node.tagorderxml.ccountrycode)',
 			PHONE = '$($InnerXML.Node.tagorderxml.cphone1)',
 			PHONE2 = '$($InnerXML.Node.tagorderxml.cphone2)',
 			ODR_DATE = '$($InnerXML.Node.tagorderxml.orddate)',
 			CARDTYPE = '$($InnerXML.Node.tagorderxml.cardtype)',
 			EXP = '$($InnerXML.Node.tagorderxml.expires)',
 			SALES_ID = '$($InnerXML.Node.tagorderxml.salesid)',
 			CTYPE2 = '$($InnerXML.Node.tagorderxml.ctype2)',
 			ENTRYDATE = '$(Get-Date -format d)',
 			EMAIL = '$($InnerXML.Node.tagorderxml.cemail)',
 			EXTENSION = '$($InnerXML.Node.tagorderxml.cphext1)',
 			EXTENSION2 = '$($InnerXML.Node.tagorderxml.cphext2)'	
 			WHERE custnum = $NewSoldToCust"
 			$Result = Invoke-Sqlcmd -Query $Query -ServerInstance $ServerInstance -Database $Database -ConnectionTimeout $ConnectionTimeout -QueryTimeout $QueryTimeout
 			
 			$Query = "
 			INSERT INTO CUSTRELA (BELONGNUM,RELA_TYPE,CUSTNUM,DESC1) VALUES ($NewSoldToCust,'C',$($InnerXML.Node.tagorderxml.ccustnum),'')
 			INSERT INTO CUSTRELA (BELONGNUM,RELA_TYPE,CUSTNUM,DESC1) VALUES ($($InnerXML.Node.tagorderxml.ccustnum),'P',$NewSoldToCust,'')"
 			$Result = Invoke-Sqlcmd -Query $Query -ServerInstance $ServerInstance -Database $Database -ConnectionTimeout $ConnectionTimeout -QueryTimeout $QueryTimeout	
 		}
 		catch [System.Exception]
 		{
 			exception_handler -Message "$($Order.Name) - Error creating new contact customer: $_" -Severity 'error' -MoveFile $Order.FullName
 			continue
 		}
 	}
 
 	########################
 	########################
 		
 	
 	try
 	{
 		$OrderXML.InnerXML | Out-File $Order.FullName -Encoding Default
 	}
 	catch [System.Exception]
 	{
 		exception_handler -Message "$($Order.Name) - Unable to write updated XML to file: $_" -Severity 'error' -MoveFile $Order.FullName
 		continue
 	}		
 
 	try
 	{
 		[string]$AOGArguments = "\\NTSERVER1\NT_Programs\AOG\OrderGen.ini $($Order.FullName) $AOGCSVOutputPath\OrderImport.txt"
 		[string]$AOGEXEPath = '\\NTSERVER1\NT_Programs\AOG\ordergen.exe'
 		$AOGProcess = Start-Process -PassThru -Wait -FilePath $AOGEXEPath -ArgumentList $AOGArguments
 				
 		
 		$Query = "SELECT TOP 1 message FROM ntImportErrors WHERE orderno = '$AltNum' ORDER BY errortime DESC" 
 		$Result = Invoke-Sqlcmd -Query $Query -ServerInstance $ServerInstance -Database $Database -ConnectionTimeout $ConnectionTimeout -QueryTimeout $QueryTimeout	
 		
 		if($Result.message -ne $null)
 		{
 			Throw $($Result.message)
 			continue
 		}
 		
 	}
 	catch [System.Exception]
 	{
 		exception_handler -Message "$($Order.Name) - AOG processing failed: $_" -Severity 'error' -AttachmentPath $($Order.FullName) -MoveFile $Order.FullName
 		continue
 	}
 	
 	$CSVheaders = @("CUSTNUM","ALTNUM","LASTNAME","FIRSTNAME","COMPANY","ADDRESS1","ADDRESS2","CITY","STATE","ZIPCODE","CFOREIGN","PHONE","COMMENT","CTYPE1","CTYPE2","CTYPE3","TAXEXEMPT","PROSPECT","CARDTYPE","CARDNUM","EXPIRES","SOURCE_KEY","CCATALOG","SALES_ID","OPER_ID","REFERENCE","SHIPVIA","FULFILLED","PAID","CONTINUED","ORDER_DATE","ODR_NUM","PRODUCT01","QUANTITY01","PRODUCT02","QUANTITY02","PRODUCT03","QUANTITY03","PRODUCT04","QUANTITY04","PRODUCT05","QUANTITY05","SLASTNAME","SFIRSTNAME","SCOMPANY","SADDRESS1","SADDRESS2","SCITY","SSTATE","SZIPCODE","HOLDDATE","PAYMETHOD","GREETING1","GREETING2","PROMOCRED","USEPRICES","PRICE01","DISCOUNT01","PRICE02","DISCOUNT02","PRICE03","DISCOUNT03","PRICE04","DISCOUNT04","PRICE05","DISCOUNT05","USESHIPAMT","SHIPPING","EMAIL","COUNTRY","SCOUNTRY","PHONE2","SPHONE","SPHONE2","SEMAIL","ORDERTYPE","INPART","TITLE","SALU","HONO","EXT","EXT2","STITLE","SSALU","SHONO","SEXT","SEXT2","SHIP_WHEN","GREETING3","GREETING4","GREETING5","GREETING6","PASSWORD","CUSTOM01","CUSTOM02","CUSTOM03","CUSTOM04","CUSTOM05","RCODE","APPROVAL","AVS","ANTRANS_ID","AUTH_AMT","AUTH_TIME","INTERNETID","ORDNOTE1","ORDNOTE2","ORDNOTE3","ORDNOTE4","ORDNOTE5","FULFILL1","FULFILL2","FULFILL3","FULFILL4","FULFILL5","INTERNET","PRIORITY","NOMAIL","NORENT","NOEMAIL","PONUMBER","ADDRESS3","SADDRESS3","CC_OTHER","PROMO_CODE","BEST_PROMO","ORDERMEMO1","ORDERMEMO2","ORDERMEMO3","MULTISHIP","R_CODE01","R_CODE02","R_CODE03","R_CODE04","R_CODE05","CARDHOLDER","ISSUE_NUM","IMPORT_ID","FRDATE","ALTIT_ID01","ALTIT_ID02","ALTIT_ID03","ALTIT_ID04","ALTIT_ID05","ROUTINGNUM","ACCOUNTNUM","ACCTTYPE","BANKNAME","SECURITYKY","PAYPALID","TX_CODE","PREAUTH","BESTTTC","NOCALL","NOFAX","CERTID","GC_RECIP","GC_EXPDATE","GC_NOTE","PTS_AMT01","PTS_USED01","PTS_AMT02","PTS_USED02","PTS_AMT03","PTS_USED03","PTS_AMT04","PTS_USED04","PTS_AMT05","PTS_USED05","SHIP_HOLD","UPDATEADDR","COMMENT2","GCCARDNUM","GCAMOUNT","GCBALANCE","GCAPPROVAL","LOGIN_ID","CUSTTOKEN","PAYTOKEN")
 	
 	try
 	{
 		$AOGCSV = Import-Csv $AOGCSVOutputPath\OrderImport.txt -Header $CSVheaders
 	}
 	catch [System.Exception]
 	{
 		exception_handler -Message "$($Order.Name) - AOG ERROR!" -Severity 'error' -AttachmentPath $($Order.FullName) -MoveFile $Order.FullName
 		continue
 	}
 	
 	#########################
 	#########################
 
 	###
 	###
 	
 	
 	if($InnerXML.Node.tagorderxml.sstate -ne '')
 	{
 		$State = $InnerXML.Node.tagorderxml.sstate
 	}
 	elseif($InnerXML.Node.tagorderxml.scustnum -ne '')
 	{
 		$Query = "SELECT RTRIM(STATE) AS 'STATE' FROM CUST WHERE custnum = '$($InnerXML.Node.tagorderxml.scustnum)'" 
 		$Result = Invoke-Sqlcmd -Query $Query -ServerInstance $ServerInstance -Database $Database -ConnectionTimeout $ConnectionTimeout -QueryTimeout $QueryTimeout	
 		if($Result.STATE.Length -eq 2)
 		{
 			$State = $Result.STATE
 		}
 	}
 	elseif($InnerXML.Node.tagorderxml.bstate -ne '')
 	{
 		$State = $InnerXML.Node.tagorderxml.bstate
 	}
 	else
 	{
 		exception_handler -Message "$($Order.Name) - Could not find state to use in tax calculation`: $_" -Severity 'warning' -MoveFile $Order.FullName
 	}
 	
 	if($State.Length -eq 2)
 	{
 		$Query = "SELECT TAXRATE FROM STATE WHERE STATE = '$State'" 
 		$Result = Invoke-Sqlcmd -Query $Query -ServerInstance $ServerInstance -Database $Database -ConnectionTimeout $ConnectionTimeout -QueryTimeout $QueryTimeout	
 	}
 	else
 	{
 		exception_handler -Message "$($Order.Name) - State value `"$State`" for tax lookup is invalid`: $_`n`n sstate`:$($InnerXML.Node.tagorderxml.sstate) `n scustnum`:$($InnerXML.Node.tagorderxml.scustnum) `n bstate`:$($InnerXML.Node.tagorderxml.bstate)" -Severity 'warning' -NoEmail
 	}
 	
 	$MOMXML = New-Object -TypeName XML
 	$MOMXML.PreserveWhitespace = $True
 	$MOMXML.Load($MOMImportXMLTemplate)
 	
 	try
 	{
 		ForEach($Record in $AOGCSV)
 		{		
 			$XMLTemplate = @($MOMXML.VFPDataSet.Import)[0]
 			$CloneNode = $XMLTemplate.Clone()		
 
 			$CloneNode.CUSTNUM		=	$Record.CUSTNUM
 			$CloneNode.ALTNUM		=	$AltNum
 			$CloneNode.LASTNAME		=	$Record.LASTNAME
 			$CloneNode.FIRSTNAME	=	$Record.FIRSTNAME
 			$CloneNode.COMPANY		=	$Record.COMPANY
 			$CloneNode.ADDRESS1		=	$Record.ADDRESS1
 			$CloneNode.ADDRESS2		=	$Record.ADDRESS2
 			$CloneNode.CITY			=	$Record.CITY
 			$CloneNode.STATE		=	$Record.STATE
 			$CloneNode.ZIPCODE		=	$Record.ZIPCODE
 			$CloneNode.CFOREIGN		=	$Record.CFOREIGN
 			$CloneNode.PHONE		=	$Record.PHONE
 			$CloneNode.COMMENT		=	$Record.COMMENT
 			$CloneNode.CTYPE1		=	$Record.CTYPE1
 			$CloneNode.CTYPE2		=	$Record.CTYPE2
 			$CloneNode.CTYPE3		=	$Record.CTYPE3
 			$CloneNode.TAXEXEMPT	=	$Record.TAXEXEMPT
 			$CloneNode.PROSPECT		=	$Record.PROSPECT
 			$CloneNode.CARDTYPE		=	$Record.CARDTYPE
 			$CloneNode.CARDNUM		=	$InnerXML.Node.tagorderxml.cardnum
 			$CloneNode.EXPIRES		=	$InnerXML.Node.tagorderxml.expires
 			$CloneNode.SOURCE_KEY	=	$Record.SOURCE_KEY
 			$CloneNode.CCATALOG		=	$Record.CCATALOG
 			$CloneNode.SALES_ID		=	$Record.SALES_ID
 			$CloneNode.OPER_ID		=	$Record.OPER_ID
 			$CloneNode.REFERENCE	=	$Record.REFERENCE
 			$CloneNode.SHIPVIA		=	$Record.SHIPVIA
 			$CloneNode.FULFILLED	=	$Record.FULFILLED
 			$CloneNode.PAID			=	if($Record.PAID -eq ''){'0.00'}else{$Record.PAID}
 			$CloneNode.CONTINUED	=	$Record.CONTINUED
 			$CloneNode.ORDER_DATE	=	if($Record.ORDER_DATE -ne ''){"$(Get-Date -Format yyyy-MM-dd -Date $Record.ORDER_DATE)"}
 			$CloneNode.ODR_NUM		=	$Record.ODR_NUM
 			$CloneNode.PRODUCT01	=	$Record.PRODUCT01
 			$CloneNode.QUANTITY01	=	if($Record.QUANTITY01 -eq ''){'0.00'}else{$Record.QUANTITY01}
 			$CloneNode.PRODUCT02	=	$Record.PRODUCT02
 			$CloneNode.QUANTITY02	=	if($Record.QUANTITY02 -eq ''){'0.00'}else{$Record.QUANTITY02}
 			$CloneNode.PRODUCT03	=	$Record.PRODUCT03
 			$CloneNode.QUANTITY03	=	if($Record.QUANTITY03 -eq ''){'0.00'}else{$Record.QUANTITY03}
 			$CloneNode.PRODUCT04	=	$Record.PRODUCT04
 			$CloneNode.QUANTITY04	=	if($Record.QUANTITY04 -eq ''){'0.00'}else{$Record.QUANTITY04}
 			$CloneNode.PRODUCT05	=	$Record.PRODUCT05
 			$CloneNode.QUANTITY05	=	if($Record.QUANTITY05 -eq ''){'0.00'}else{$Record.QUANTITY05}
 			$CloneNode.SLASTNAME	=	$Record.SLASTNAME
 			$CloneNode.SFIRSTNAME	=	$Record.SFIRSTNAME
 			$CloneNode.SCOMPANY		=	$Record.SCOMPANY
 			$CloneNode.SADDRESS1	=	$Record.SADDRESS1
 			$CloneNode.SADDRESS2	=	$Record.SADDRESS2
 			$CloneNode.SCITY		=	$Record.SCITY
 			$CloneNode.SSTATE		=	$Record.SSTATE
 			$CloneNode.SZIPCODE		=	$Record.SZIPCODE
 			$CloneNode.PAYMETHOD 	=	$InnerXML.Node.tagorderxml.paymethod
 			$CloneNode.GREETING1	=	$Record.GREETING1
 			$CloneNode.GREETING2	=	$Record.GREETING2
 			$CloneNode.PROMOCRED	=	if($Record.PROMOCRED -eq ''){'0.00'}else{$Record.PROMOCRED}
 			$CloneNode.USEPRICES	=	$Record.USEPRICES
 			$CloneNode.PRICE01		=	if($Record.PRICE01 -eq ''){'0.00'}else{$Record.PRICE01}
 			$CloneNode.DISCOUNT01	=	if($Record.DISCOUNT01 -eq ''){'0'}else{$Record.DISCOUNT01.Substring(0,$Record.DISCOUNT01.IndexOf("."))}
 			$CloneNode.PRICE02		=	if($Record.PRICE02 -eq ''){'0.00'}else{$Record.PRICE02}
 			$CloneNode.DISCOUNT02	=	if($Record.DISCOUNT02 -eq ''){'0'}else{$Record.DISCOUNT02.Substring(0,$Record.DISCOUNT02.IndexOf("."))}
 			$CloneNode.PRICE03		=	if($Record.PRICE03 -eq ''){'0.00'}else{$Record.PRICE03}
 			$CloneNode.DISCOUNT03	=	if($Record.DISCOUNT03 -eq ''){'0'}else{$Record.DISCOUNT03.Substring(0,$Record.DISCOUNT03.IndexOf("."))}
 			$CloneNode.PRICE04		=	if($Record.PRICE04 -eq ''){'0.00'}else{$Record.PRICE04}
 			$CloneNode.DISCOUNT04	=	if($Record.DISCOUNT04 -eq ''){'0'}else{$Record.DISCOUNT04.Substring(0,$Record.DISCOUNT04.IndexOf("."))}
 			$CloneNode.PRICE05		=	if($Record.PRICE05 -eq ''){'0.00'}else{$Record.PRICE05}
 			$CloneNode.DISCOUNT05	=	if($Record.DISCOUNT05 -eq ''){'0'}else{$Record.DISCOUNT05.Substring(0,$Record.DISCOUNT05.IndexOf("."))}
 			$CloneNode.USESHIPAMT	=	$Record.USESHIPAMT
 			$CloneNode.SHIPPING		=	if($Record.SHIPPING -eq ''){'0.00'}else{$Record.SHIPPING}
 			$CloneNode.EMAIL		=	$Record.EMAIL
 			$CloneNode.COUNTRY		=	$Record.COUNTRY
 			$CloneNode.SCOUNTRY		=	$Record.SCOUNTRY
 			$CloneNode.PHONE2		=	$Record.PHONE2
 			$CloneNode.SPHONE		=	$Record.SPHONE
 			$CloneNode.SPHONE2		=	$Record.SPHONE2
 			$CloneNode.SEMAIL		=	$Record.SEMAIL
 			$CloneNode.ORDERTYPE	=	$Record.ORDERTYPE
 			$CloneNode.INPART		=	$Record.INPART
 			$CloneNode.TITLE		=	$Record.TITLE
 			$CloneNode.SALU			=	$Record.SALU
 			$CloneNode.HONO			=	$Record.HONO
 			$CloneNode.EXT			=	$Record.EXT
 			$CloneNode.EXT2			=	$Record.EXT2
 			$CloneNode.STITLE		=	$Record.STITLE
 			$CloneNode.SSALU		=	$Record.SSALU
 			$CloneNode.SHONO		=	$Record.SHONO
 			$CloneNode.SEXT			=	$Record.SEXT
 			$CloneNode.SEXT2		=	$Record.SEXT2
 			$CloneNode.GREETING3	=	$Record.GREETING3
 			$CloneNode.GREETING4	=	$Record.GREETING4
 			$CloneNode.GREETING5	=	$Record.GREETING5
 			$CloneNode.GREETING6	=	$Record.GREETING6
 			$CloneNode.PASSWORD		=	$Record.PASSWORD
 			#$CloneNode.CUSTOM02	=	$Record.CUSTOM02
 			#$CloneNode.CUSTOM03	=	$Record.CUSTOM03
 			#$CloneNode.CUSTOM04	=	$Record.CUSTOM04
 			#$CloneNode.CUSTOM05	=	$Record.CUSTOM05
 			$CloneNode.RCODE 		=	$InnerXML.Node.tagorderxml.rcode
 			$CloneNode.APPROVAL 	=	$InnerXML.Node.tagorderxml.approval
 			$CloneNode.AVS 			=	$InnerXML.Node.tagorderxml.avs
 			$CloneNode.ANTRANS_ID	=	$InnerXML.Node.tagorderxml.antrans_id
 			$CloneNode.AUTH_AMT 	=	if($InnerXML.Node.tagorderxml.auth_amt -IsNot [int]){'0'}else{$InnerXML.Node.tagorderxml.auth_amt}
 			$CloneNode.AUTH_TIME 	=	$InnerXML.Node.tagorderxml.auth_time
 			$CloneNode.INTERNETID	=	$Record.INTERNETID
 			#$CloneNode.ORDNOTE2	=	$Record.ORDNOTE2
 			#$CloneNode.ORDNOTE3	=	$Record.ORDNOTE3
 			#$CloneNode.ORDNOTE4	=	$Record.ORDNOTE4
 			#$CloneNode.ORDNOTE5	=	$Record.ORDNOTE5
 			#$CloneNode.FULFILL2	=	$Record.FULFILL2
 			#$CloneNode.FULFILL3	=	$Record.FULFILL3
 			#$CloneNode.FULFILL4	=	$Record.FULFILL4
 			#$CloneNode.FULFILL5	=	$Record.FULFILL5
 			$CloneNode.INTERNET		=	if($Record.INTERNET -eq 'T'){'true'}elseif($Record.INTERNET -eq 'F'){'false'}else{'false'}
 			$CloneNode.PRIORITY		=	$Record.PRIORITY
 			$CloneNode.NOMAIL		=	if($Record.NOMAIL -eq ''){'false'}else{$Record.NOMAIL}
 			$CloneNode.NORENT		=	if($Record.NORENT -eq ''){'false'}else{$Record.NORENT}
 			$CloneNode.NOEMAIL		=	if($Record.NOEMAIL -eq ''){'false'}else{$Record.NOEMAIL}
 			$CloneNode.PONUMBER		=	$Record.PONUMBER
 			$CloneNode.ADDRESS3		=	$Record.ADDRESS3
 			$CloneNode.SADDRESS3	=	$Record.SADDRESS3
 			$CloneNode.CC_OTHER		=	$Record.CC_OTHER
 			$CloneNode.PROMO_CODE	=	$Record.PROMO_CODE
 			$CloneNode.BEST_PROMO	=	if($Record.BEST_PROMO -eq ''){'false'}else{$Record.BEST_PROMO}
 			$CloneNode.ORDERMEMO1	=	$Record.ORDERMEMO1
 			$CloneNode.ORDERMEMO2	=	$Record.ORDERMEMO2
 			$CloneNode.ORDERMEMO3	=	$Record.ORDERMEMO3
 			$CloneNode.R_CODE01		=	$Record.R_CODE01
 			$CloneNode.R_CODE02		=	$Record.R_CODE02
 			$CloneNode.R_CODE03		=	$Record.R_CODE03
 			$CloneNode.R_CODE04		=	$Record.R_CODE04
 			$CloneNode.R_CODE05		=	$Record.R_CODE05
 			$CloneNode.CARDHOLDER 	=	$InnerXML.Node.tagorderxml.cardholder
 			$CloneNode.ISSUE_NUM	=	$Record.ISSUE_NUM
 			$CloneNode.FRDATE		=	''
 			$CloneNode.ALTIT_ID01	=	''
 			$CloneNode.ALTIT_ID02	=	''
 			$CloneNode.ALTIT_ID03	=	''
 			$CloneNode.ALTIT_ID04	=	''
 			$CloneNode.ALTIT_ID05	=	''
 			$CloneNode.ROUTINGNUM	=	''
 			$CloneNode.ACCOUNTNUM	=	''
 			$CloneNode.ACCTTYPE		=	''
 			$CloneNode.BANKNAME		=	''
 			$CloneNode.SECURITYKY	=	''
 			$CloneNode.PAYPALID		=	''
 			$CloneNode.TX_CODE		=	''
 			$CloneNode.PREAUTH		=	'false'
 			$CloneNode.BESTTTC		=	''
 			$CloneNode.NOCALL		=	'false'
 			$CloneNode.NOFAX		=	'false'
 			$CloneNode.CERTID		=	'0'
 			$CloneNode.GC_RECIP		=	''
 			$CloneNode.GC_NOTE		=	''
 			$CloneNode.PTS_AMT01	=	'0.00'
 			$CloneNode.PTS_USED01	=	'false'
 			$CloneNode.PTS_AMT02	=	'0.00'
 			$CloneNode.PTS_USED02	=	'false'
 			$CloneNode.PTS_AMT03	=	'0.00'
 			$CloneNode.PTS_USED03	=	'false'
 			$CloneNode.PTS_AMT04	=	'0.00'
 			$CloneNode.PTS_USED04	=	'false'
 			$CloneNode.PTS_AMT05	=	'0.00'
 			$CloneNode.PTS_USED05	=	'false'
 			$CloneNode.UPDATEADDR	=	'false'
 			$CloneNode.COMMENT2		=	''
 			$CloneNode.GCCARDNUM	=	''
 			$CloneNode.GCAMOUNT		=	'0.00'
 			$CloneNode.GCBALANCE	=	'0.00'
 			$CloneNode.GCAPPROVAL	=	''
 			$CloneNode.LOGIN_ID 	=	''
 			$CloneNode.CUSTTOKEN	=	$InnerXML.Node.tagorderxml.custtoken
 			$CloneNode.PAYTOKEN		=	$InnerXML.Node.tagorderxml.paytoken
 			$CloneNode.STAX			=	if($TaxRate -ne $null){"$TaxRate"}else{'0.00'}
 			$CloneNode.USETAXAMT	=	'1'
 			
 			$MOMXML.DocumentElement.AppendChild($CloneNode) | Out-Null
 		}
 	}
 	catch [System.Exception]
 	{
 		exception_handler -Message "CSV -> MOMXML mapping failed: $_" -Severity 'error' -MoveFile $Order.FullName
 		continue
 	}
 	
 	$MOMXML.VFPDataSet.RemoveChild($MOMXML.VFPDataSet.Import[0]) | Out-Null
 	
 	$MOMXML.Save($MOMXMLOutputFile)
 	
 	##################
 	
 	$NewOrderNum = $MOM_API.OrderImport("$MOMXMLOutputFile", "$MOMImportSchemaPath", 1, $false, "", "", $null)
 	
 	if(-not ($NewOrderNum -gt 0))
 	{
 		Throw exception_handler -Message "$($Order.Name) - API Order Import failed! AOG#: $($Record.ODR_NUM) " -Severity 'error'
 		continue
 	}
 	
 	Write-Verbose "Order Import Succeeded. Order#: $NewOrderNum"
 	
 	###################
 	###################
 	
 	###
 	###
 	
 	try
 	{
 		if($InnerXML.Node.tagorderxml.quote -eq 'true')
 		{
 			$Query = "UPDATE CMS SET quotation = 1, order_st2 =  'QO' WHERE ORDERNO = $NewOrderNum"
 			$Result = Invoke-Sqlcmd -Query $Query -ServerInstance $ServerInstance -Database $Database -ConnectionTimeout $ConnectionTimeout -QueryTimeout $QueryTimeout
 		}
 	}
 	catch
 	{
 		continue
 	}
 	
 	try
 	{
 		$Query = "UPDATE CMS SET soldnum = "
 		
 		if($InnerXML.Node.tagorderxml.xml_type -eq 'OLR' -and $InnerXML.Node.tagorderxml.ccustnum -ne ''){
 			$Query += $InnerXML.Node.tagorderxml.ccustnum}
 		elseif($InnerXML.Node.tagorderxml.xml_type -eq 'SC_New' -and $InnerXML.Node.tagorderxml.bcustnum -ne '') {
 			$Query += $InnerXML.Node.tagorderxml.bcustnum}
 		elseif($NewSoldToCust -ne $null) {
 			$Query += $NewSoldToCust}
 		else {
 			Throw "Neither <[bc]custnum> or a new custnum is populated or bad xml_type."}
 		
 		$Query += " WHERE ORDERNO = $NewOrderNum"
 		$Result = Invoke-Sqlcmd -Query $Query -ServerInstance $ServerInstance -Database $Database -ConnectionTimeout $ConnectionTimeout -QueryTimeout $QueryTimeout
 	}
 	catch
 	{
 		continue
 	}
 	
 	try
 	{
 
 		$Query = "UPDATE CMS SET SHIPNUM = "
 		
 		if($InnerXML.Node.tagorderxml.scustnum -ne '') {
 			$Query += $InnerXML.Node.tagorderxml.scustnum}
 		elseif($NewShipCust -ne $null) {
 			$Query += $NewShipCust}
 		else {
 			Throw "Neither <scustnum> or a new scustnum is populatede."}
 		
 		$Query += " WHERE ORDERNO = $NewOrderNum"
 		$Result = Invoke-Sqlcmd -Query $Query -ServerInstance $ServerInstance -Database $Database -ConnectionTimeout $ConnectionTimeout -QueryTimeout $QueryTimeout
 	}
 	catch
 	{
 		continue
 	}
 	
 	try
 	{	
 		if($InnerXML.Node.tagorderxml.baddress1 -ne '' -and $InnerXML.Node.tagorderxml.bcity -ne '' -and $InnerXML.Node.tagorderxml.bstate -ne '' -and $InnerXML.Node.tagorderxml.bzip.Length -ge 5)
 		{		
 			$type = UPSValidate-Address `
 			-Address $InnerXML.Node.tagorderxml.baddress1 `
 			-City $InnerXML.Node.tagorderxml.bcity `
 			-State $InnerXML.Node.tagorderxml.bstate `
 			-Zip $(if($InnerXML.Node.tagorderxml.bzip.Length -eq 5){$InnerXML.Node.tagorderxml.bzip}) `
 			-ZipExt $(if($InnerXML.Node.tagorderxml.bzip.Length -ge 9){$InnerXML.Node.tagorderxml.bzip.Substring($InnerXML.Node.tagorderxml.bzip.IndexOf('-')+1,4)})
 		}
 		elseif($InnerXML.Node.tagorderxml.saddress1 -ne '' -and $InnerXML.Node.tagorderxml.scity -ne '' -and $InnerXML.Node.tagorderxml.sstate -ne '' -and $InnerXML.Node.tagorderxml.szip.Length -ge 5)
 		{
 			$type = UPSValidate-Address `
 			-Address $InnerXML.Node.tagorderxml.saddress1 `
 			-City $InnerXML.Node.tagorderxml.scity `
 			-State $InnerXML.Node.tagorderxml.sstate `
 			-Zip $(if($InnerXML.Node.tagorderxml.szip.Length -eq 5){$InnerXML.Node.tagorderxml.szip}) `
 			-ZipExt $(if($InnerXML.Node.tagorderxml.szip.Length -ge 9){$InnerXML.Node.tagorderxml.szip.Substring($InnerXML.Node.tagorderxml.szip.IndexOf('-')+1,4)})
 		}
 		else
 		{
 			$type = 'invalid'
 		}	
 		
 		if($type -ne 'Residential')
 		{
 			$Query = "UPDATE CUST SET upscomdelv = 1 WHERE custnum IN ( $($InnerXML.Node.tagorderxml.scustnum),$($InnerXML.Node.tagorderxml.bcustnum) )"
 			$Result = Invoke-Sqlcmd -Query $Query -ServerInstance $ServerInstance -Database $Database -ConnectionTimeout $ConnectionTimeout -QueryTimeout $QueryTimeout
 		}
 		
 		switch ($type)
 		{
 			'invalid'	{
 							
 							$Query = "UPDATE DEMGRAPH SET colUPS_INVALID = 1 WHERE custnum IN ( $($InnerXML.Node.tagorderxml.scustnum),$($InnerXML.Node.tagorderxml.bcustnum) )"
 							$Result = Invoke-Sqlcmd -Query $Query -ServerInstance $ServerInstance -Database $Database -ConnectionTimeout $ConnectionTimeout -QueryTimeout $QueryTimeout
 							Write-Verbose "$NewOrderNum - Set colups_invalid = 1."
 						}
 			'Unknown'	{
 							$Query = "UPDATE DEMGRAPH SET colUPS_UNKNOWN = 1 WHERE custnum IN ( $($InnerXML.Node.tagorderxml.scustnum),$($InnerXML.Node.tagorderxml.bcustnum) )"
 							$Result = Invoke-Sqlcmd -Query $Query -ServerInstance $ServerInstance -Database $Database -ConnectionTimeout $ConnectionTimeout -QueryTimeout $QueryTimeout
 							Write-Verbose "$NewOrderNum - Set colups_unknown = 1."
 						}
 		}
 	}
 	catch
 	{
 	}
 	
 	try
 	{
 		$Query = "
 		UPDATE CMS SET ODR_DATE = '$($InnerXML.Node.tagorderxml.orddate.Replace('T',' '))' --This replace pulls the letter T (for the time section) out of the XML.
 		WHERE CMS.ORDERNO = $NewOrderNum"
 		$Result = Invoke-Sqlcmd -Query $Query -ServerInstance $ServerInstance -Database $Database -ConnectionTimeout $ConnectionTimeout -QueryTimeout $QueryTimeout
 		Write-Verbose "CMS.ODR_DATE updated from <orddate> node."
 	}
 	catch [System.Exception]
 	{
 	}
 	
 	
 	try
 	{
 		if($InnerXML.Node.predefined.process -ne "")
 		{
 			$Query = "UPDATE ntCustpredefines SET MomOrderNum = $NewOrderNum WHERE AOGNum = $AltNum"
 			$Result = Invoke-Sqlcmd -Query $Query -ServerInstance $ServerInstance -Database $Database -ConnectionTimeout $ConnectionTimeout -QueryTimeout $QueryTimeout
 		}
 	}
 	catch
 	{
 		continue
 	}
 	
 	try
 	{
 		$Query = "UPDATE ntorderidgen SET newmomnum = $NewOrderNum WHERE intnetordnum = $AltNum"
 		$Result = @(Invoke-Sqlcmd -Query $Query -ServerInstance $ServerInstance -Database $Database -ConnectionTimeout $ConnectionTimeout -QueryTimeout $QueryTimeout)
 	}
 	catch
 	{
 		continue
 	}	
 	
 	try
 	{
 		$Query = "
 		IF NOT EXISTS (SELECT ORDERNO FROM ORDMEMO WHERE ORDERNO = $NewOrderNum)	
 		BEGIN
 			INSERT INTO ORDMEMO 
 			([ORDERNO],[NOTES],[FULFILL],[DESC1],[DESC2],[DESC3],[DESC4],[DESC5],[DESC6],[ITEMFULFIL])
 			VALUES ($NewOrderNum,'','','','','','','','','')
 		END"
 		$Result = Invoke-Sqlcmd -Query $Query -ServerInstance $ServerInstance -Database $Database -ConnectionTimeout $ConnectionTimeout -QueryTimeout $QueryTimeout
 	}
 	catch [System.Exception]
 	{
 	}
 	
 	try
 	{		
 		$Query = "
 		--Check to see if contact info exist in ntOrderIDGen
 		IF EXISTS (	SELECT SoldToFName, SoldToLName, SoldToPhone, SoldToPExt 
 					FROM ntOrderIDGen
 					WHERE
 					ISNULL(SoldToFName,'') <> '' AND 
 					ISNULL(SoldToLName,'') <> '' AND  
 					ISNULL(SoldToPhone,'') <> '' AND
 					newmomnum = $NewOrderNum )
 			BEGIN 
 				(SELECT SoldToFName, SoldToLName, SoldToPhone, SoldToPExt 
 				FROM ntOrderIDGen WHERE newmomnum = $NewOrderNum)
 			END
 		ELSE
 			BEGIN
 				--No contact info in ntOrderIDGen check to see if SoldToCust is available
 				IF EXISTS (	SELECT SoldToCust FROM ntOrderIDGen
 							WHERE SoldToCust > 0 AND
 							newmomnum = $NewOrderNum)
 					BEGIN
 						(SELECT FIRSTNAME, LASTNAME, PHONE, EXTENSION FROM CUST
 						 WHERE CUSTNUM =	(SELECT TOP 1 SoldToCust FROM ntOrderIDGen
                                              WHERE newmomnum = $NewOrderNum ORDER BY OrderGenID DESC)
 						)
 					END
 				ELSE
 					BEGIN
 						--SoldToCust is not populated, find contact info using CustNum
 						(SELECT FIRSTNAME, LASTNAME, PHONE, EXTENSION FROM CUST
 						 WHERE CUSTNUM =	(SELECT TOP 1 Custnum FROM ntOrderIDGen
                                              WHERE newmomnum = $NewOrderNum ORDER BY OrderGenID DESC)
 						)
 					END
 			END"
 		$CallDetails = Invoke-Sqlcmd -Query $Query -ServerInstance $ServerInstance -Database $Database -ConnectionTimeout $ConnectionTimeout -QueryTimeout $QueryTimeout
 	
 		$Query = "
 		UPDATE ORDMEMO 
 		SET
 		--AOG fixed notes
 		ORDMEMO.NOTES = CAST(ntOrderIDGen.ordernotes AS VARCHAR(MAX)) + CHAR(13) + CHAR(10) + CHAR(13) + CHAR(10) +
 		--Online notes
 		'$($InnerXML.Node.tagorderxml.ordnotes)' + CHAR(13) + CHAR(10) + CHAR(13) + CHAR(10)"
 		if($InnerXML.Node.tagorderxml.call -eq 'True' -and $CallDetails.FIRSTNAME.Trim() -ne '' -and $CallDetails.LASTNAME.Trim() -ne '' -and $CallDetails.PHONE.Trim() -ne '')
 		{
 			$Query += "
 			--Call notes
 			+ '$(if($InnerXML.Node.tagorderxml.call -eq 'True'){`"Call for credit card info: $($CallDetails.FIRSTNAME.Trim()) $($CallDetails.LASTNAME.Trim()) - $($CallDetails.PHONE.Trim()) $($CallDetails.EXTENSION.Trim())`"})'" 
 		}
 		else
 		{
 			Write-Verbose "One or more call details (ntOrderIDGen.SoldTo*) fields are blank!"
 		}
 		$Query += "
 		--AOG fixed fulfill
 		,ORDMEMO.FULFILL = ntOrderIDGen.ffnotes
 		FROM ORDMEMO
 		INNER JOIN ntOrderIDGen ON ORDMEMO.ORDERNO = ntOrderIDGen.newmomnum
 		WHERE ORDMEMO.ORDERNO = $NewOrderNum"
 	
 		$Result = Invoke-Sqlcmd -Query $Query -ServerInstance $ServerInstance -Database $Database -ConnectionTimeout $ConnectionTimeout -QueryTimeout $QueryTimeout
 	}
 	catch [System.Exception]
 	{
 		continue
 	}
 	
 	
 	try
 	{
 		$Query = "
 		UPDATE ITEMS
 		SET ITEMS.CUSTOMINFO = n.custominfo
 		FROM ITEMS i
 		INNER JOIN CMS c
 		ON i.ORDERNO = c.ORDERNO
 		INNER JOIN NTMOMPRODUCTS n
 		ON c.ALT_ORDER = n.orderid
 		WHERE i.ORDERNO = $NewOrderNum
 		AND	i.SEQ = n.seqnumber"
 
 		$Result = Invoke-Sqlcmd -Query $Query -ServerInstance $ServerInstance -Database $Database -ConnectionTimeout $ConnectionTimeout -QueryTimeout $QueryTimeout
 	}
 	catch [System.Exception]
 	{
 		exception_handler -Message "Unable to update ITEMS.CUSTOMINFO record(s) for orderno $NewOrderNum`: $_" -Severity 'error' -MoveFile $Order.FullName
 		continue
 	}
 	
 	#############
 	#############
 		
 	Write-Verbose "$($Order.Name) - Import successful. Latest Company: $($InnerXML.node.tagorderxml.bcompany) Order#: $NewOrderNum"
 	exception_handler -Message "$($Order.Name) - Import successful. Company: $($InnerXML.node.tagorderxml.bcompany) Order#: $NewOrderNum" -Severity 'info' -NoEmail
 	
 	if($NoDeleteLocal -eq $false)
 	{
 		try
 		{
 			Write-Verbose "NoDelete Switch is not present. Deleting Order: $Order"
 			Remove-Item $Order.FullName
 		}
 		catch [System.Exception]
 		{
 			exception_handler -Message "Unable to delete order XML file: $_" -Severity 'error'
 		}
 	}
 	
 	#########################
 	#########################
 	
 	Write-Verbose "`n--- MOVE FILES FROM OLD ---"
 	
 	try
 	{
 		$old_folder = Get-ChildItem $OldCustomersPath | Where-Object {$_.Name -like "$($Record.CUSTNUM)[ _]*" -and $_.PSIsContainer -eq $true}
 		
 		if($old_folder.Count -eq 1)
 		{
 			$Query = "
 			UPDATE DEMGRAPH SET 
 			COLPIMAGE = REPLACE(COLPIMAGE,'_Old\',''),
 			COLPDF = REPLACE(COLPDF,'_Old\','')
 			WHERE CUSTNUM IN	(SELECT BELONGNUM FROM CUSTRELA 
 								WHERE CUSTNUM  = $($Record.CUSTNUM) AND 
 								RELA_TYPE = 'C')
 				  				AND COLPIMAGE <> ''"
 			$Result = Invoke-Sqlcmd -Query $Query -ServerInstance $ServerInstance -Database $Database -ConnectionTimeout $ConnectionTimeout -QueryTimeout $QueryTimeout
 			
 			Move-Item -Path $old_folder.FullName -Destination $CurrentCustomersPath -Force
 			
 			Write-Verbose "Moved customer folder and updated proof path for: $($old_folder.Name)"
 		}
 		elseif($old_folder.Count -gt 1)
 		{
 			Throw 'Two folders share the same customer number. Fix and then move manually.'
 		}
 		else
 		{
 			Write-Verbose "Customer folder is not in _Old, no relocation will occur."
 		}
 	}
 	catch [System.Exception]
 	{
 		 exception_handler -Message "Unable to move folder`: $_" -Severity 'error' -NoEmail
 	}
 	
 
 if((Get-Variable MOM_API -ErrorAction SilentlyContinue) -ne $null)
 {
 	$MOM_API.DBDisconnect() | Out-Null
 	Remove-Variable MOM_API
 }
`

