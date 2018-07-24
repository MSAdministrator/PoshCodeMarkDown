---
Author: jan egil ring
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 1995
Published Date: 
Archived Date: 2010-08-05t23
---

# set-outlooksignature.ps1 - set-outlooksignature.ps1

## Description

create outlook signature based on user information from active directory. settings for versioning, template-files, enforcing for new and replyforward are stored in the registry.

## Comments

script to create an outlook signature based on user information from active directory.

## Usage



## TODO



## script

``

## Code

`#
 #
 ###########################################################################"
 #
 #
 #
 #
 #
 #
 #
 ###########################################################################"
 
 $CompanyName = 'Company Name'
 $DomainName = 'domain.local'
 $SigSource = "\\$DomainName\netlogon\sig_files\$CompanyName"
 
 $AppData=(Get-Item env:appdata).value
 $SigPath = '\Microsoft\Signatures'
 $LocalSignaturePath = $AppData+$SigPath
 
 $UserName = $env:username
 $Filter = "(&(objectCategory=User)(samAccountName=$UserName))"
 $Searcher = New-Object System.DirectoryServices.DirectorySearcher
 $Searcher.Filter = $Filter
 $ADUserPath = $Searcher.FindOne()
 $ADUser = $ADUserPath.GetDirectoryEntry()
 $ADDisplayName = $ADUser.DisplayName
 $ADEmailAddress = $ADUser.mail
 $ADTitle = $ADUser.title
 $ADTelePhoneNumber = $ADUser.TelephoneNumber
 
 $CompanyRegPath = "HKCU:\Software\"+$CompanyName
 
 if (Test-Path $CompanyRegPath)
 {}
 else
 {New-Item -path "HKCU:\Software" -name $CompanyName}
 
 if (Test-Path $CompanyRegPath'\Outlook Signature Settings')
 {}
 else
 {New-Item -path $CompanyRegPath -name "Outlook Signature Settings"}
 
 $ForcedSignatureNew = (Get-ItemProperty $CompanyRegPath'\Outlook Signature Settings').ForcedSignatureNew
 $ForcedSignatureReplyForward = (Get-ItemProperty $CompanyRegPath'\Outlook Signature Settings').ForcedSignatureReplyForward
 $SignatureVersion = (Get-ItemProperty $CompanyRegPath'\Outlook Signature Settings').SignatureVersion
 Set-ItemProperty $CompanyRegPath'\Outlook Signature Settings' -name SignatureSourceFiles -Value $SigSource
 $SignatureSourceFiles = (Get-ItemProperty $CompanyRegPath'\Outlook Signature Settings').SignatureSourceFiles
 
 
 if ($ForcedSignatureNew -eq '1')
 {
 $MSWord = New-Object -com word.application
 $EmailOptions = $MSWord.EmailOptions
 $EmailSignature = $EmailOptions.EmailSignature
 $EmailSignatureEntries = $EmailSignature.EmailSignatureEntries
 $EmailSignature.NewMessageSignature=$CompanyName
 $MSWord.Quit()
 }
 
 if ($ForcedSignatureReplyForward -eq '1')
 {
 $MSWord = New-Object -com word.application
 $EmailOptions = $MSWord.EmailOptions
 $EmailSignature = $EmailOptions.EmailSignature
 $EmailSignatureEntries = $EmailSignature.EmailSignatureEntries
 $EmailSignature.ReplyMessageSignature=$CompanyName
 $MSWord.Quit()
 }
 
 if ($SignatureVersion -eq $SigVersion){}
 else
 {
 Copy-Item "$SignatureSourceFiles\*" $LocalSignaturePath -Recurse -Force
 
 $MSWord = New-Object -com word.application
 $MSWord.Documents.Open($LocalSignaturePath+'\'+$CompanyName+'.rtf')
 ($MSWord.ActiveDocument.Bookmarks.Item("DisplayName")).Select()
 $MSWord.Selection.Text=$ADDisplayName
 ($MSWord.ActiveDocument.Bookmarks.Item("Title")).Select()
 $MSWord.Selection.Text=$ADTitle
 ($MSWord.ActiveDocument.Bookmarks.Item("TelephoneNumber")).Select()
 $MSWord.Selection.Text=$ADTelePhoneNumber
 ($MSWord.ActiveDocument.Bookmarks.Item("EmailAddress")).Select()
 $MSWord.Selection.Text=$ADEmailAddress
 ($MSWord.ActiveDocument).Save()
 ($MSWord.ActiveDocument).Close()
 $MSWord.Quit()
 
 $MSWord = New-Object -com word.application
 $MSWord.Documents.Open($LocalSignaturePath+'\'+$CompanyName+'.htm')
 ($MSWord.ActiveDocument.Bookmarks.Item("DisplayName")).Select()
 $MSWord.Selection.Text=$ADDisplayName
 ($MSWord.ActiveDocument.Bookmarks.Item("Title")).Select()
 $MSWord.Selection.Text=$ADTitle
 ($MSWord.ActiveDocument.Bookmarks.Item("TelephoneNumber")).Select()
 $MSWord.Selection.Text=$ADTelePhoneNumber
 ($MSWord.ActiveDocument.Bookmarks.Item("EmailAddress")).Select()
 $MSWord.Selection.Text=$ADEmailAddress
 ($MSWord.ActiveDocument).Save()
 ($MSWord.ActiveDocument).Close()
 $MSWord.Quit()
 
 
 (Get-Content $LocalSignaturePath'\'$CompanyName'.txt') | Foreach-Object {$_ -replace "DisplayName", $ADDisplayName -replace "Title", $ADTitle -replace "TelePhoneNumber", $ADTelePhoneNumber -replace "EmailAddress", $ADEmailAddress} | Set-Content $LocalSignaturePath'\'$CompanyName'.txt'
 }
 
 
 if ($ForcedSignatureNew -eq $ForceSignatureNew){}
 else
 {Set-ItemProperty $CompanyRegPath'\Outlook Signature Settings' -name ForcedSignatureNew -Value $ForceSignatureNew}
 
 if ($ForcedSignatureReplyForward -eq $ForceSignatureReplyForward){}
 else
 {Set-ItemProperty $CompanyRegPath'\Outlook Signature Settings' -name ForcedSignatureReplyForward -Value $ForceSignatureReplyForward}
 
 if ($SignatureVersion -eq $SigVersion){}
 else
 {Set-ItemProperty $CompanyRegPath'\Outlook Signature Settings' -name SignatureVersion -Value $SigVersion}
`

