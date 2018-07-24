---
Author: ravikanth_chaganti
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 2255
Published Date: 
Archived Date: 2010-09-26t20
---

# poshcode ise addon - 

## Description

this is the final version of poshcode ise addon for uploading scripts to poshcode.org from powershell ise. you can upload scripts in two ways. first method is to select all or part of the script and upload it by pressing ctrl+alt+c. and, the second method is to just press ctrl+alt+c. this way you can upload all contents without any selection.

## Comments



## Usage



## TODO



## module

`get-poshcodepreferences`

## Code

`#
 #
 if (!(Get-Module WPK)) {Import-Module -global WPK}
 if (!(Get-Module PoshCode)) {Import-Module -global PoshCode}
 
 Function Get-PoshCodePreferences {
     if (Get-Item $global:xmlPath -ErrorAction SilentlyContinue) {
         try {
             $pcPreferences = Import-Clixml -Path $global:xmlPath
             $global:PCSyntax = $pcPreferences.Syntax
             $global:PCExpiry = $pcPreferences.Expiry
             $global:PCAuthor = $pcPreferences.Author
         }
         catch {
             Write-Host "Import-CliXml failed with following error"
             return
         }
     } else {
         $global:PCSyntax = "posh"
         $global:PCExpiry = "forever"
         $global:PCAuthor = "$($env:USERNAME)"
     }
 }
 
 Function Save-PoshCodePreferences {
     param($syntax, $expiry, $author)
     
     $pcPreferences = New-Object PSObject
     $pcPreferences | Add-Member -MemberType NoteProperty -Name "Syntax" -Value $syntax
     $pcPreferences | Add-Member -MemberType NoteProperty -Name "Expiry" -Value $expiry
     $pcPreferences | Add-Member -MemberType NoteProperty -Name "Author" -Value $author
     
     try {
         Export-Clixml -InputObject $pcPreferences -Path $global:xmlPath -Force
     }
     catch {
         Write-Host "Export-Clixml; failed with the following error"
         Write-Host $error[0].InnerException
         return
     }
     [system.Windows.Forms.MessageBox]::show('Your preferences have been saved')
 }
 
 function Show-PoshCodeGUI {
     Get-PoshCodePreferences
     New-Window -Title "PoshCode addon" -WindowStartupLocation CenterScreen -Width 836 -Height 477 -Show -ResizeMode NoResize -On_Loaded {
         $txtPasteCode     = $Window | Get-ChildControl txtPasteCode
         $cmbSyntax        = $Window | Get-ChildControl cmbSyntax
         $cmbExpiry        = $Window | Get-ChildControl cmbExpiry
         $txtTitle         = $window | Get-ChildControl txtTitle
         $txtDescription   = $window | Get-ChildControl txtDescription
         $txtAuthor        = $window | Get-ChildControl txtAuthor
 		$btnSavePref	  = $window | Get-ChildControl btnSavePref
 		$btnSubmit	 	  = $window | Get-ChildControl btnSubmit
     } {
             New-Grid {
                 New-TextBox -Name txtPasteCode -Margin "12,12,0,0" -Height 258 -Width 800 `
                             -HorizontalAlignment "Left" -VerticalAlignment "Top" -Text $global:SelectedText `
                             -IsReadOnly -VerticalScrollBarVisibility "Auto" -HorizontalScrollBarVisibility "Auto"
                         
                 New-Label -Name lblLang -Margin "12,281,0,0" -Height 28 `
                             -HorizontalAlignment "Left" -VerticalAlignment "Top" -Content "Scripting Language" -FontWeight "Bold"
             
                 New-ComboBox -Name cmbSyntax -Margin "175,285,0,0" -Height 23 -Width 192 `
                             -HorizontalAlignment "Left" -VerticalAlignment "Top" -Items $global:cmbSyntaxOptions -SelectedIndex $global:cmbSyntaxOptions.IndexOf($global:PCSyntax)
                 
                 New-Label -Name lblExpiry -Margin "12,321,0,0" -Height 28 `
                             -HorizontalAlignment "Left" -VerticalAlignment "Top" -Content "Keep" -FontWeight "Bold"
             
                 New-ComboBox -Name cmbExpiry -Margin "175,324,0,0" -Height 23 -Width 192 `
                             -HorizontalAlignment "Left" -VerticalAlignment "Top" -Items $global:Expiry -SelectedIndex $global:Expiry.IndexOf($global:PCExpiry)
                         
                 New-Label -Name lblTitle -Margin "450,281,0,0" -Height 28 `
                             -HorizontalAlignment "Left" -VerticalAlignment "Top" -Content "Title" -FontWeight "Bold"
             
                 New-TextBox -Name txtTitle -Margin "570,285,0,0" -Height 23 -Width 192 `
                             -HorizontalAlignment "Left" -VerticalAlignment "Top"
                                     
                 New-Label -Name lblAuthor -Margin "450,321,0,0" -Height 28 `
                             -HorizontalAlignment "Left" -VerticalAlignment "Top" -Content "Author" -FontWeight "Bold"
             
                 New-TextBox -Name txtAuthor -Margin "570,324,0,0" -Height 23 -Width 192 `
                             -HorizontalAlignment "Left" -VerticalAlignment "Top" -Text $global:PCAuthor
 
                 New-Label -Name lblDescription -Margin "12,361,0,0" -Height 28 `
                             -HorizontalAlignment "Left" -VerticalAlignment "Top" -Content "Description" -FontWeight "Bold"
                         
                 New-TextBox -Name txtDescription -Margin "175,361,0,0" -Height 35 -Width 590 `
                             -HorizontalAlignment "Left" -VerticalAlignment "Top"                           
             
                 New-Button -Name btnSavePref -Margin "280,400,0,0" -Height 23 -Width 110 `
                             -HorizontalAlignment "Left" -VerticalAlignment "Top" -Content "Save Preferences" -On_Click {
 								Save-PoshCodePreferences -Syntax $cmbSyntax.SelectedItem -Expiry $cmbExpiry.SelectedItem -Author $txtAuthor.Text
                             }
             
                 New-Button -Name btnSubmit -Margin "410,400,0,0" -Height 23 -Width 110 `
                             -HorizontalAlignment "Left" -VerticalAlignment "Top" -Content "Submit to PoshCode" -On_Click {
                                 $returnUri = $global:SelectedText | New-PoshCode -title  $txtTitle.Text `
                                    	-description $txtDescription.Text -Author $txtAuthor.Text `
                                     -keep $cmbExpiry.SelectedItem -Language $cmbSyntax.SelectedItem
                                 if ($returnUri) {
                                     [System.Diagnostics.Process]::Start($returnUri)
                                     $window.Close()
                                 } else {
                                     [system.Windows.Forms.MessageBox]::show('Error occured while uploading to PoshCode')
                                     $window.Close()
                                 }
 							}
         }
     }
 }
 
 [System.Collections.ArrayList]$global:cmbSyntaxOptions = "posh","text","vbnet","xml","asp","bash","csharp"
 [System.Collections.ArrayList]$global:Expiry = "forever","day","month"
 
 $global:xmlPath = $("$env:APPDATA\PoshCodePrefs.xml")
 
 if ($host.Name â��eq 'Windows PowerShell ISE Host') {
     $scriptBlock = {
         if (($psISE.CurrentFile.Editor.SelectedText -ne "")) {
             $global:SelectedText = $psISE.CurrentFile.Editor.SelectedText
         } elseif (($psISE.CurrentFile.Editor.Text -ne "")) {
                 $global:SelectedText = $psISE.CurrentFile.Editor.Text
         } else {
             [system.Windows.Forms.MessageBox]::show('There is nothing to upload. Select some text or open a script')
             return
         }
         Show-PoshCodeGUI
     }
     
     $submenus = $psise.CurrentPowerShellTab.AddOnsMenu.Submenus
     $menuExists = $false
     
     foreach ($menuItem in $subMenus) {
         if ($menuItem.DisplayName -eq "Send to Posh_Code") {
             $menuExists = $true
         }
     }
     
     if (!$menuExists) {
         $psISE.CurrentPowerShellTab.AddOnsMenu.SubMenus.Add("Send To Posh_Code",$ScriptBlock,"CTRL+ALT+C")
     }
     
     $ExecutionContext.SessionState.Module.OnRemove = {
         foreach ($menuItem in $subMenus) {
             if ($menuItem.DisplayName -eq "Send to PPosh_Code") {
                 $submenus.Remove($menuItem)
                 return
             }
         }
     }
 } else {
     Write-Host "This module is meant for either ISE or PGSE"
     return
 }
 
 Export-ModuleMember -Function * -Variable *
`

