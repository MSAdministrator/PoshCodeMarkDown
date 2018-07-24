---
Author: kevin
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1632
Published Date: 2010-02-08t20
Archived Date: 2016-11-28t20
---

# find-duplicatesmtp - 

## Description

this script will query your ad for all users, groups and contacts.  it will then process through each of the smtp addresses for all objects and count the number of occurances of each address.  the scope expanded as i kept thinking of other interesting questions regarding the smtp addresses that were in my domain.  once the script has finished gathering interesting data regarding the smtp addresses in your domain, it will dump that information out to a spreadsheet (requires excel), then send the spreadsheet as an email to you.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #######################################################################
 #######################################################################
 
 
 $SMTPServer = "127.0.0.1" 
 $SenderAddress = "FromAddress@yourdomain.com"
 $RecipientAddresses = "You@yourdomain.com"
 $Subject = "Duplicate SMTP Address Report"
 $Domain = "yourdomain.com"
 
 #$ScriptPath = "C:\Scripts\"
 $ScriptPath = ($myInvocation.MyCommand.Path).Replace($myInvocation.MyCommand.Name, "")
 
 
 if ((Get-PSSnapin "Quest.ActiveRoles.ADManagement" -ErrorAction SilentlyContinue) -eq $null) 
 {
 	Add-PSSnapin "Quest.ActiveRoles.ADManagement" -ErrorVariable Err -ErrorAction SilentlyContinue
 	if ($Err) 
 	{
 		Write-Host "`tError loading Quest.ActiveRoles.ADManagement" -ForeGroundColor Green
 		exit
 	}
 } 
 
 Write-Host "`n`tGathering email addresses from Active Directory" -ForeGroundColor Yellow
 $Filter = '(|(&(objectClass=user)(homeMDB=*))(&(mailNickName=*)(objectClass=Contact))(&(mailNickName=*)(objectClass=group)))'
 $MailObjects = get-qadobject -service $Domain -sizeLimit 0 -ldapfilter $Filter -DontUseDefaultIncludedProperties `
 	-IncludedProperties proxyAddresses | Select proxyAddresses, ClassName
 $NumberMailobjects = $MailObjects.count
 Write-Host "`tFound " -noNewline -ForeGroundColor Yellow
 Write-Host "$NumberMailobjects " -noNewLine -ForeGroundColor Green
 Write-Host "mail enabled objects" -ForeGroundColor Yellow
 
 Write-Host "`n`tCounting proxyAddresses Data" -ForeGroundColor Yellow
 $EmailCount = @{}
 $EmailTypeCount = @{}
 $EmailDomainCount = @{}
 $oldPos = $host.UI.RawUI.CursorPosition
 Foreach ($MailObject in $MailObjects){
 	$ObjectType = $MailObject.ClassName
 	$MailObject.ProxyAddresses | Foreach {
 		$Type = [string]$_.split(":")[0]
 		if($EmailTypeCount.ContainsKey($Type) -eq $False)
 		{
 			$EmailTypeCount.Add($Type, 1)
 		}
 		Else
 		{
 			$Count = $EmailTypeCount.Get_Item($Type)
 			$Count++
 			$EmailTypeCount.Set_Item($Type, $Count)
 		}
 		
 		if($EmailCount.ContainsKey($_) -eq $False)
 		{
 			$EmailCount.Add($_, 1)
 		}
 		Else
 		{
 			$Count = $EmailCount.Get_Item($_)
 			$Count++
 			$EmailCount.Set_Item($_, $Count)
 		}
 		
 		$Domain = [string]$_.Split("@")[1]
 		if($Domain -ne $null)
 		{
 			if($EmailDomainCount.ContainsKey($Domain) -eq $False)
 			{
 				$EmailDomainCount.Add($Domain, 1)
 			}
 			Else
 			{
 				$Count = $EmailDomainCount.Get_Item($Domain)
 				$Count++
 				$EmailDomainCount.Set_Item($Domain, $Count)
 			}
 		}
 	}
 	$UpdateOutPut = $False
 	If ($NumberAddType -lt $EmailTypeCount.Count){$UpdateOutPut = $True}
 	elseIf ($NumberMailDomains -lt $EmailDomainCount.Count){$UpdateOutPut = $True}
 	elseIf (($EmailCount.Count % 100) -eq 0){$UpdateOutPut = $True}
 	
 	If ($UpdateOutPut -eq $True)
 	{
 		$NumberAddType = $EmailTypeCount.Count
 		$NumberAddresses = $EmailCount.Count
 		$NumberMailDomains = $EmailDomainCount.Count
 		$host.UI.RawUI.CursorPosition = $oldPos
 		Write-Host "`tFound " -noNewline -ForeGroundColor Yellow
 		Write-Host "$NumberAddresses " -noNewLine -ForeGroundColor Green
 		Write-Host "unique email addresses" -ForeGroundColor Yellow
 		Write-Host "`tFound " -noNewline -ForeGroundColor Yellow
 		Write-Host "$NumberAddType " -noNewLine -ForeGroundColor Green
 		Write-Host "address types" -ForeGroundColor Yellow
 		Write-Host "`tFound " -noNewline -ForeGroundColor Yellow
 		Write-Host "$NumberMailDomains " -noNewLine -ForeGroundColor Green
 		Write-Host "mail domains`n" -ForeGroundColor Yellow
 	}
 }
 $EmailDomains = $EmailDomainCount.GetEnumerator() | Sort Key
 $EmailTypes = $EmailTypeCount.GetEnumerator() | Sort Key
 
 
 Write-Host "`n`tFiltering for duplicate email addresses" -ForeGroundColor Yellow
 $Duplicates = $EmailCount.GetEnumerator() | ? {$_.Value -gt 1}
 $Duplicates  = $Duplicates | Sort
 $NumberDuplicates = $Duplicates.Count
 Write-Host "`tFound " -noNewline -ForeGroundColor Yellow
 Write-Host "$NumberDuplicates " -noNewLine -ForeGroundColor Green
 Write-Host "duplicate email addresses`n" -ForeGroundColor Yellow
 
 
 Write-Host "`n`tGathering information about the objects with duplicate email addresses" -ForeGroundColor Yellow
 $DupeOutput = @()
 $Count = 0
 $oldPos = $host.UI.RawUI.CursorPosition
 $Duplicates | Foreach {
 	$count++
 	[string]$Email = $_.Key
 	$Filter = "(proxyAddresses=*$Email*)"
 	$ObjectsWithDupes = get-qadobject -service $Domain -ldapFilter $Filter `
 		-DontUseDefaultIncludedProperties -includedProperties extensionAttribute3 | `
 		Select DisplayName, samAccountName, DN, ClassName, extensionAttribute3
 	$ObjectsWithDupes | foreach {
 		$_ | add-member noteproperty -Name "DupeEmailAddress" -Value $Email
 	}
 	$DupeOutput += $ObjectsWithDupes
 	$DupesProcessed = ($DupeOutput | Select DN -Unique).Count
 	$UsersProcessed = ($DupeOutput | ? {$_.Classname -eq "user"} | Select DN -Unique).Count
 	$GroupsProcessed = ($DupeOutput | ? {$_.Classname -eq "group"} | Select DN -Unique).Count
 	$ContactsProcessed = ($DupeOutput | ? {$_.Classname -eq "contact"} | Select DN -Unique).Count
 	$host.UI.RawUI.CursorPosition = $oldPos
 	Write-Host "`tProcessed " -noNewline -ForeGroundColor Yellow
 	Write-Host "$DupesProcessed " -noNewLine -ForeGroundColor Green
 	Write-Host "objects and " -noNewLine -ForeGroundColor Yellow
 	Write-Host "$Count" -noNewLine -ForeGroundColor Green
 	Write-Host " addresses" -ForeGroundColor Yellow
 }
 $DupeCount = $DupeOutput.count
 $DupeOutput = $DupeOutput | Sort displayname, ClassName, DupeEmailAddress
 
 Write-Host "`n`tGenerating spreadsheet" -ForeGroundColor Yellow
 $Excel = New-Object -comobject Excel.Application
 $Excel.Visible = $False
 $WB = $Excel.Workbooks.Add(1)
 
 $EmailParseData = $WB.Worksheets.Item(1)
 [void]$EmailParseData.Activate()
 $EmailParseData.Name = "SMTP Data"
 
 
 [void]$Excel.Cells.Item(2,1).Select()
 $Excel.ActiveWindow.FreezePanes = $True
 [void]$Excel.Range($Excel.Cells.item((1),(1)),$Excel.cells.item((1),(2))).Select()
 $Excel.Selection.Interior.ColorIndex = 48
 [void]$Excel.Selection.Font.Bold
 $Excel.Selection.Font.Size = 12
 $Excel.Selection.HorizontalAlignment = -4108
 [void]$Excel.Range($Excel.Cells.item((1),(4)),$Excel.cells.item((1),(5))).Select()
 $Excel.Selection.Interior.ColorIndex = 48
 [void]$Excel.Selection.Font.Bold
 $Excel.Selection.Font.Size = 12
 $Excel.Selection.HorizontalAlignment = -4108
 $Row = 1
 
 $EmailParseData.Cells.Item($Row,1) = "Domain"
 $Excel.Columns.item("A:A").ColumnWidth = 45
 $EmailParseData.Cells.Item($Row,2) = "Number Of Occurances"
 $Excel.Columns.item("B:B").ColumnWidth = 23
 $EmailParseData.Cells.Item($Row,4) = "Address Type"
 $Excel.Columns.item("D:D").ColumnWidth = 16
 $EmailParseData.Cells.Item($Row,5) = "Number Of Occurances"
 $Excel.Columns.item("E:E").ColumnWidth = 23
 $Row = 2
 
 Write-Host "`n`tWriting Email Domains" -ForeGroundColor Yellow
 $EmailDomains | Foreach {
 	$EmailParseData.Cells.Item($Row,1) = $_.Key
 	$EmailParseData.Cells.Item($Row,2) = $_.Value
 	$Row++
 }
 
 Write-Host "`n`tWriting Address Types" -ForeGroundColor Yellow
 $Row = 2
 $EmailTypes | Foreach {
 	$EmailParseData.Cells.Item($Row,4) = $_.Key
 	$EmailParseData.Cells.Item($Row,5) = $_.Value
 	$Row++
 }
 
 Write-Host "`n`t`Creating Second Worksheet" -ForeGroundColor Yellow
 $DupeWS = $Excel.Worksheets.Add()
 [void]$DupeWS.Activate()
 $DupeWS.Name = "Duplicate Address Data"
 $Row = 1
 
 [void]$Excel.Cells.Item(2,1).Select()
 $Excel.ActiveWindow.FreezePanes = $True
 [void]$Excel.Range($Excel.Cells.item((1),(1)),$Excel.cells.item((1),(6))).Select()
 $Excel.Selection.Interior.ColorIndex = 48
 [void]$Excel.Selection.Font.Bold
 $Excel.Selection.Font.Size = 12
 $Excel.Selection.HorizontalAlignment = -4108
 
 $DupeWS.Cells.Item($row,1) = "DisplayName"
 $Excel.Columns.item("A:A").ColumnWidth = 35
 
 $DupeWS.Cells.Item($row,2) = "samAccountName"
 $Excel.Columns.item("B:B").ColumnWidth = 25
 
 $DupeWS.Cells.Item($row,3) = "DupeEmailAddress"
 $Excel.Columns.item("C:C").ColumnWidth = 60
 
 $DupeWS.Cells.Item($row,4) = "ClassName"
 $Excel.Columns.item("D:D").ColumnWidth = 20
 
 $DupeWS.Cells.Item($row,5) = "ExtensionAttribute3"
 $Excel.Columns.item("E:E").ColumnWidth = 35
 
 $DupeWS.Cells.Item($row,6) = "DN"
 $Excel.Columns.item("F:F").ColumnWidth = 90
 
 $row++
 Write-Host "`n`tWriting Duplicate Address Data" -ForeGroundColor Yellow
 $oldPos = $host.UI.RawUI.CursorPosition
 $DupeOutput | Foreach {
 	$DupeWS.Cells.Item($row,1) = $_.Displayname
 	$DupeWS.Cells.Item($row,2) = $_.Samaccountname
 	$DupeWS.Cells.Item($row,3) = $_.DupeEmailAddress
 	$DupeWS.Cells.Item($row,4) = $_.ClassName
 	$DupeWS.Cells.Item($row,5) = $_.extensionAttribute3
 	$DupeWS.Cells.Item($row,6) = $_.DN
 	$row++
 	
 	If (($row % 5) -eq 0)
 	{
 		$host.UI.RawUI.CursorPosition = $oldPos
 		Write-Host "`tOutput " -noNewline -ForeGroundColor Yellow
 		Write-Host "$row " -noNewLine -ForeGroundColor Green
 		Write-Host "lines to Excel`n" -ForeGroundColor Yellow
 	}
 }
 $host.UI.RawUI.CursorPosition = $oldPos
 Write-Host "`tOutput " -noNewline -ForeGroundColor Yellow
 Write-Host "$row " -noNewLine -ForeGroundColor Green
 Write-Host "lines to Excel" -ForeGroundColor Yellow
 
 $Excel.DisplayAlerts = $False
 $saveAs = $ScriptPath + "DupeEmailReport." + (get-date).dayofyear + ".xls"
 write-host "`tSaving Report to: $saveAS`n`n`n" -ForegroundColor Cyan
 $WB.SaveAs($saveAs, 1)
 $WB.Close()
 $Excel.Quit()
 
 $Body = "`<BR>" +`
 		"  Mail Enabled Objects Found`t`t $NumberMailobjects " +`
 		"`<BR>" +`
 		"  Unique Email Addresses Found:`t`t $NumberAddresses " + `
 		"`<BR>" + `
 		"  Duplicated Email Addresses:`t`t $NumberDuplicates " + `
 		"`<BR><BR>" + `
 		"  Mail Objects Affected:`t`t $DupesProcessed " + `
 		"`<BR>" + `
 		"  Users Affected:`t`t $UsersProcessed " + `
 		"`<BR>" + `
 		"  Groups Affected:`t`t $GroupsProcessed " + `
 		"`<BR>" + `
 		"  Contacts Affected:`t`t $ContactsProcessed " + `
 		"`<BR><BR>" +`
 		"  Number of Address Types:`t`t $NumberAddType " + `
 		"`<BR>" + `
 		"  Number of Mail Domains:`t`t $NumberMailDomains" 
 		
 $Attachment = new-object System.Net.Mail.Attachment($saveAs)
 $objMail = new-object System.Net.Mail.MailMessage
 $objMail.From = $SenderAddress
 $objMail.Sender = $SenderAddress
 $objMail.To.Add($RecipientAddresses)
 $objMail.Subject = $Subject
 $objMail.Body = $Body
 $objMail.IsBodyHTML = $true
 $objMail.Attachments.Add($Attachment)
 
 $objSMTP = New-Object System.Net.Mail.SmtpClient
 $objSMTP.Host = $SMTPServer
 $objSMTP.UseDefaultCredentials = $true
 
 $objSMTP.Send($objMail)
`

