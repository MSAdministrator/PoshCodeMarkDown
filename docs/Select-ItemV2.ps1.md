---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 8.25
Encoding: ascii
License: cc0
PoshCode ID: 4342
Published Date: 
Archived Date: 2016-11-11t11
---

#  - 

## Description

dot sourced psise snippet editor form

## Comments



## Usage



## TODO



## function

`select-itemv2`

## Code

`#
 #
 function Select-ItemV2 {
     PARAM (
         [Parameter(Mandatory=$true, ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$true,Position=0)]
         $options,
         [Parameter(Mandatory=$true, ValueFromPipeline=$True, ValueFromPipelineByPropertyName=$true, Position=1)]
         $displayProperty,
         [Parameter(Mandatory=$false, ValueFromPipeline=$True, ValueFromPipelineByPropertyName=$true)]
         [string]$title="Select an item from the list",
         [Parameter(Mandatory=$false, ValueFromPipeline=$True, ValueFromPipelineByPropertyName=$true,
         HelpMessage='Form Size format is an Array of two interger values as: @(width, height)`nMinimum form width: 225, minimum form height: 250!')]
         [ValidateRange(225,1050)]
         [array]$PopupFormSize
     )
     
     [Windows.Forms.form]$PopupForm = new-object Windows.Forms.form
     [System.Windows.Forms.ListBox]$lstOptions = New-Object System.Windows.Forms.ListBox
     [System.Windows.Forms.Button]$btnOK = New-Object System.Windows.Forms.Button 
     [System.Windows.Forms.Button]$btnCancel = New-Object System.Windows.Forms.Button
     Add-Member -InputObject $PopupForm -MemberType NoteProperty -Name gv_ReturnItem -Value $null
     $PopupForm.StartPosition = "CenterParent" #"CenterScreen"
     $PopupForm.ControlBox = $False
     $PopupForm.KeyPreview = $True
     $PopupForm.Add_KeyDown({if ($_.KeyCode -eq "Enter") {if (($btnOk.Enabled)) {& $btnOkClick}}})
     $PopupForm.Add_KeyDown({if ($_.KeyCode -eq "Escape") {& $btnCancelClick}})
     
     $btnOkClick = {if ($lstOptions.SelectedIndex -lt 0) {
             $PopupForm.gv_ReturnItem = $null
         } else {
             $PopupForm.gv_ReturnItem = $options[$lstOptions.SelectedIndex]
             $PopupForm.Close()
         }
     }
     $btnCancelClick = {$PopupForm.gv_ReturnItem = $null;$PopupForm.Close()}
     $lstOptionsChanged = {if ($lstOptions.SelectedIndex -ge 0) {$btnOk.Enabled = $true} else {$btnOk.Enabled = $false} }
     
     if ($PopupFormSize) {
         if ($PopupFormsize[0] -lt 225) {$PopupFormsize[0] = 225}
         if ($PopupFormsize[1] -lt 250) {$PopupFormsize[1] = 250}
     } else {
         $PopupFormsize = @(225,250)
     }
     $lstsize = @(($PopupFormsize[0]-25),($PopupFormsize[1]-70))
     $btnOksize = @([int](($PopupFormsize[0]/2)-100),($PopupFormsize[1]-65))
     $btnCancelsize = @([int](($PopupFormsize[0]/2)+5),($PopupFormsize[1]-65))
     $PopupForm.Size = new-object System.Drawing.Size @($PopupFormsize[0],$PopupFormsize[1])   
     $PopupForm.text = $title
                       
     $lstOptions.Name = "lstOptions"
     $lstOptions.Width = $lstsize[0]
     $lstOptions.Height = $lstsize[1]
     $lstOptions.Location = New-Object System.Drawing.Size(5,5)
     $lstOptions.Add_SelectedIndexChanged($lstOptionsChanged)
     $lstOptions.Add_MouseDoubleClick($btnOkClick)
                   
     $btnOK.Width=75
     $btnOK.Location = New-Object System.Drawing.Size($btnOksize[0], $btnOksize[1])
     $btnOK.Text = "OK"
     $btnOK.add_click($btnOkClick)
     $btnOk.Enabled = $false
     $PopupForm.Controls.Add($lstOptions)
     $PopupForm.Controls.Add($btnOK)
                   
     $btnCancel.Width=75
     $btnCancel.Location = New-Object System.Drawing.Size($btnCancelsize[0], $btnCancelsize[1])
     $btnCancel.Text = "Cancel"
     $btnCancel.Add_Click($btnCancelClick)
     $PopupForm.Controls.Add($btnCancel)
                       
     foreach ($option in $options) {
         $lstOptions.Items.Add($option.$displayProperty) | Out-Null
     }
     $PopupForm.ShowDialog() | Out-Null
     return ($PopupForm.gv_ReturnItem)
 }
 
 cls
 [System.Windows.Forms.Application]::EnableVisualStyles()
 [Windows.Forms.form]$form = new-object Windows.Forms.form
 $form.Size = new-object System.Drawing.Size (933, 575)
 $form.StartPosition = "CenterParent" #"CenterScreen"  
 $form.text = "PowerShellISE Snippet Editor: by "
 $form.BackColor = "lightgray"
 $form.MinimizeBox = $False
 $form.MaximizeBox = $False
 $form.WindowState = "Normal"
 
 $lblTitle = new-object System.Windows.Forms.Label
 $txtTitle = new-object System.Windows.Forms.TextBox
 $lblAuthor = new-object System.Windows.Forms.Label
 $txtAuthor = new-object System.Windows.Forms.TextBox
 $lblDescription = new-object System.Windows.Forms.Label
 $txtDescription = new-object System.Windows.Forms.TextBox
 $lblScriptCode = new-object System.Windows.Forms.Label
 $txtScriptCode = new-object System.Windows.Forms.TextBox
 $lblLanguage = new-object System.Windows.Forms.Label
 $txtLanguage = new-object System.Windows.Forms.TextBox
 $lblExpansion = new-object System.Windows.Forms.Label
 $txtSnippetType = new-object System.Windows.Forms.TextBox
 $lblCursorOffset = new-object System.Windows.Forms.Label
 $txtCursorOffset = new-object System.Windows.Forms.TextBox
 $lblVarOffset = new-object System.Windows.Forms.Label
 $txtVarOffset = new-object System.Windows.Forms.TextBox
 $lblCaretOffset = new-object System.Windows.Forms.Label
 $txtCaretOffset = new-object System.Windows.Forms.TextBox
 $lblCodeEndOffset = new-object System.Windows.Forms.Label
 $txtCodeEndOffset = new-object System.Windows.Forms.TextBox
 $lblFilePath = new-object System.Windows.Forms.Label
 $txtFilePath = new-object System.Windows.Forms.TextBox
 $lblStatus = new-object System.Windows.Forms.Label
 $ckbAutoEdit = New-Object System.Windows.Forms.CheckBox
 $ckbNoBlanks = New-Object System.Windows.Forms.CheckBox
 $btnPsIse = new-object System.Windows.Forms.Button
 $btnOpen = new-object System.Windows.Forms.Button
 $btnNew = new-object System.Windows.Forms.Button
 $btnEdit = new-object System.Windows.Forms.Button
 $btnFixOffset = new-object System.Windows.Forms.Button
 $rdbFirstVar = New-Object System.Windows.Forms.RadioButton
 $rdbCodeEnd = New-Object System.Windows.Forms.RadioButton
 $rdbCursorPos = New-Object System.Windows.Forms.RadioButton
 $rdbFirstElipsis = New-Object System.Windows.Forms.RadioButton
 $btnSave = new-object System.Windows.Forms.Button
 
 Add-Member -InputObject $form -MemberType NoteProperty -Name gv_ProgramAuthor -Value 'Larry Coffey' -TypeName [string]
 Add-Member -InputObject $form -MemberType NoteProperty -Name gv_SnippetAuthor -Value 'Larry Coffey'  -TypeName [string]
 Add-Member -InputObject $form -MemberType NoteProperty -Name gv_FileElipsisOffset -Value 0 -TypeName [int32]
 Add-Member -InputObject $form -MemberType NoteProperty -Name gv_FileVariableOffset -Value 0 -TypeName [int32]
 Add-Member -InputObject $form -MemberType NoteProperty -Name gv_FileCursorOffset -Value 0  -TypeName [int32]
 Add-Member -InputObject $form -MemberType NoteProperty -Name gv_FileCodeEndOffset -Value 1 -TypeName [int32]
 Add-Member -InputObject $form -MemberType NoteProperty -Name gv_FileLastWriteTime -Value $null
 Add-Member -InputObject $form -MemberType NoteProperty -Name gv_SnippetModified -Value $false -TypeName [bool]
 Add-Member -InputObject $form -MemberType NoteProperty -Name gv_SaveSnippetFlag -Value $false -TypeName [bool]
 
 $scriptFldr = Split-Path -parent $MyInvocation.MyCommand.Definition
 $configPath = join-path -Path $scriptFldr -ChildPath "SnippetEditorConfig.txt"
 
 $LoadAuthor = {
   if (!(Test-Path $configPath)) {
     out-file -FilePath $configPath -InputObject $form.gv_SnippetAuthor -Encoding string
   }
   [array]$configs =  Get-Content -Path $configPath
   $form.gv_SnippetAuthor = $configs[0]
 }
 
 $SaveAuthor = {
  Set-Content -Path $configPath -Value $form.gv_SnippetAuthor -Encoding String -Force
 }
 
 $ValidateAuthor = {
   if (!($txtAuthor.ReadOnly) -and ($txtAuthor.TabStop)) {
     $form.KeyPreview = $false
     $txtAuthor.ReadOnly = $true
     $txtAuthor.TabStop = $false
     $txtAuthor.ResetBackColor()
     if (($txtAuthor.Text.Trim()) -match "^\w*\s+\w*" ) {
       $form.gv_SnippetAuthor = ($txtAuthor.Text.Trim())
       & $SaveAuthor
       $lblStatus.Text = (($form.gv_SnippetAuthor) + ", your name has been permanently saved for future use!`n")
     } else {$lblStatus.Text = ("Your name was NOT entered correctly, and has reverted to the previous entry!`nAuthor: " + ($form.gv_SnippetAuthor))}
     $txtAuthor.Text = ($txtAuthor.TempAuthor)
   }
 }
 
 $ChangeDefaultAuthor = {
   $txtAuthor.Add_LostFocus($ValidateAuthor)
   $txtAuthor.TempAuthor = ($txtAuthor.Text)
   $lblStatus.Text = "Enter your first and last name into the Author TextBox.`nPress the Enter ot Tab key to Save your name or Escape to cancel Saving your name!"
   $txtAuthor.Text = $null
   $txtAuthor.ReadOnly = $false
   $txtAuthor.BackColor = 0xffC0ffC0
   $txtAuthor.TabStop = $true
   $form.KeyPreview = $true
   $txtAuthor.Focus()
 }
 
 & $LoadAuthor
 
 $Paths = $env:PSModulePath.Split(";")
 $Path = $paths[0]
 $parentArray = $Path.Split("\")
 $parentPath = "" 
 for($i=0;$i -lt (($parentArray.Count)-1);$i++) {$parentPath = $parentPath + $parentArray[($i)] + "\"}
 $snippetPath = Join-Path -Path $parentPath -ChildPath Snippets
 $snippetWild = Join-Path -Path $parentPath -ChildPath Snippets\*.ps1xml
 Write-Warning ("Hi " + ($form.gv_SnippetAuthor) + ", your snippet path is...`n '" + $snippetPath + "'")
 $form.gv_SnippetModified = $false
 $form.gv_SaveSnippetFlag = $false
 $form.Tag = $null 
 
 function Check_UnsavedSnippet($form) {
   if (($form.gv_SnippetModified) -and ($txtTitle.Text)) {
     $FileName = ($txtFilePath.Text).Split("\")[-1] 
     $msg = ("Save your changes to this snippet file?`n`nTitle: " + ($txtTitle.Text) + "`n`nFileName: " + $FileName)
     $answ = [windows.forms.messagebox]::show($msg,"SAVE SNIPPET CHANGES:",[system.windows.forms.messageboxbuttons]::yesno,[System.Windows.Forms.MessageBoxIcon]::Question)
     if ($answ -eq "Yes") {& $btnSaveClick}
   }
 }
 
 function RadioButton_CheckedChanged( $rdbObject ){ 
  if (($rdbObject.Checked)) {
    switch ($rdbObject.Name) {
      ($rdbFirstElipsis.Name) {
        if (($txtScriptCode.Elipsis1Pos) -gt 0) {
          $lblStatus.Text = ("First Elipsis found at position " + ($form.gv_FileElipsisOffset) + ".")
          $lblVarOffset.Text = "1st Elipsis:"
          $txtVarOffset.Text = ($form.gv_FileElipsisOffset)
          $txtScriptCode.SelectionStart = ($txtScriptCode.Elipsis1Pos)
          $txtScriptCode.Focus()
        } else {
          $lblStatus.Text = "Elipsis not found in snippet code."
        }
      }
      ($rdbFirstVar.Name) {
        if (($txtScriptCode.Variable1Pos) -gt 0) {
          $lblStatus.Text = ("First Variable found at position " + ($form.gv_FileVariableOffset) + ".")
          $lblVarOffset.Text = "1st Variable:"
          $txtVarOffset.Text = ($form.gv_FileVariableOffset)
          $txtScriptCode.SelectionStart = ($txtScriptCode.Variable1Pos)
          $txtScriptCode.Focus()
        } else {
         $lblStatus.Text = "Variable not found in snippet code."
        }
      }
      ($rdbCursorPos.Name) {
        if (($txtScriptCode.CursorPos) -gt 0) {
          $lblStatus.Text = ("Cursor position to be set at location " + ($form.gv_FileCursorOffset) + ".")
          $txtScriptCode.SelectionStart = ($txtScriptCode.CursorPos)
          $txtScriptCode.Focus()
        } else {
          $lblStatus.Text = "Cursor Offset at start of snippet code."
        }
      }
      ($rdbCodeEnd.Name) {
        if (($txtScriptCode.CodeEndPos) -gt 0) {
          $lblStatus.Text = ("Cursor position to be set at end of script code, at location " + ($txtCodeEndOffset.Text) + ".")
          $txtScriptCode.SelectionStart = ($txtScriptCode.CodeEndPos)
          $txtScriptCode.Focus()
        } else {
          $lblStatus.Text = "Cursor Offset at end of snippet script code."
        }
      }
    }
   }
 }
 
 function Get-CodeOffset ($selStart) {
   if ($selStart -eq 0) {
     $OF = 0
   } else {
     $OL = ($txtScriptCode.GetLineFromCharIndex($selStart))
     $OF = ($selStart - $OL)
   }
   return $OF
 }
 
 function Get-CodeLine ($selStart) {
   if ($selStart -eq 0) {
     $OL = 0
   } else {
     $OL = ($txtScriptCode.GetLineFromCharIndex($selStart))
   }
   return $OL
 }
 
 $form.Text = ($form.Text + $form.gv_ProgramAuthor)
 $btnPsIseClick = {$lblStatus.Text="Select a snippet file to edit in the PowershellISE"
   $snippets = Get-ChildItem -File -Path $snippetWild
   $ht = @{options=$snippets;displayProperty="Name";title="Select one of the following PowershellISE Snippets";formSize=@(450,500)}
   if ($ReturnedItem) {
   Write-Warning "Opening: $ReturnedItem in PSISE"
   $psISE.CurrentPowerShellTab.Files.Add(($ReturnedItem.FullName))
   $form.Close()
   } else {$lblStatus.Text="No valid snippet file path provided for PowershellISE..."}
 }
 $btnOpenClick = {$lblStatus.Text="Select a snippet file to edit in the Snippet Editer"
   $snippets = Get-ChildItem -File -Path $snippetWild
   $ht = @{options=$snippets;displayProperty="Name";title="Select one of the following PowershellISE Snippets";PopupFormSize=@(450,500)}
   if ($ReturnedItem) {$txtFilePath.Text = $ReturnedItem.FullName
     & $nullEdits
     & $loadXml
   } else {$lblStatus.Text="No valid snippet file path provided for editing..."}
 }
 $do_btnNew = {
   if (!($txtTitle.Modified) -and !($txtDescription.Modified) -and !($txtScriptCode.Modified)) {
     $form.Tag  = "Only the Title, Description and ScriptCode fields are enabled for a new snippet! Filename generated upon leaving the Title field."
     $txtFilePath.Text=$null
     $txtCaretOffset.Text = 1
     ($form.gv_FileCursorOffset) = 0
     $txtCursorOffset.Text = 0
     $form.gv_SnippetModified = $false
     & $nullEdits
     & $enableEdits
     $md = (Get-Date)
     $mds = "{0:MMM-dd-yyyy}" -f $md
     $form.gv_FileLastWriteTime = $mds
     $txtDescription.Text = ([char]27 + " ; Modified: " + $mds + [char]29)
     $txtTitle.Focus()
   } else {
     $lblStatus.Text="Unable to clear fields for a new snippet as you have unsaved work in progress!"
   }
 }
 $loadXml = {[xml]$snippetXml = Get-Content ($txtFilePath.Text)
   $txtTitle.Text = $snippetXml.Snippets.Snippet.Header.Title
   $txtDescription.Text = $snippetXml.Snippets.Snippet.Header.Description
   $txtAuthor.Text = $snippetXml.Snippets.Snippet.Header.Author
   $txtSnippetType.Text = $snippetXml.Snippets.Snippet.Header.SnippetTypes.SnippetType
   $txtLanguage.Text = $snippetXml.Snippets.Snippet.Code.Script.GetAttribute("Language")
   $txtCaretOffset.Text = $snippetXml.Snippets.Snippet.Code.Script.GetAttribute("CaretOffset")
   $txtCursorOffset.Text = $txtCaretOffset.Text
   $form.gv_FileCursorOffset = $txtCursorOffset.Text
   $txtCodeEndOffset.Text = "0"
   $txtScriptCode.CursorPos = -1
   $form.gv_SnippetModified = $false
   if (($ckbNoBlanks.Checked)) {
     $tmptxt = $snippetXml.Snippets.Snippet.Code.Script.InnerText
     if (($tmptxt.indexOf("`r`n`r`n") -ge 0)) {$tmptxt = ($tmptxt.Replace("`r`n`r`n","`r`n"))}
     if (($tmptxt.indexOf("`n`n") -ge 0)) {$tmptxt = ($tmptxt.Replace("`n`n","`n"))}
   } else {
     $tmptxt = $snippetXml.Snippets.Snippet.Code.Script.InnerText
     if (($tmptxt.indexOf("`r`n") -ge 0)) {$tmptxt = ($tmptxt.Replace("`r`n","`n"))}
   }
   $txtScriptCode.Text = $tmptxt.Replace("`n","`r`n")
   $btnFixOffset.Enabled = $false
   $form.gv_FileLastWriteTime = (Get-Item ($txtFilePath.Text)).LastWriteTime.DateTime
   & $UpdateDescription
   if ($ckbAutoEdit.Checked) {
     $lblStatus.Text=("CaretOffset fixing is only enabled before editing. (File CaretOffset: " + ($txtCaretOffset.Text) + ", Script CodeEndOffset: " + ($txtcodeEndOffset.Text) + ", " + ($lblVarOffset.Text) + " " + ($txtVarOffset.Text) +")`nDisable AutoEdit as the time to fix it is just after snippet opening, or use the save button when done editing!")
     & $enableEdits
     $txtScriptCode.SelectionStart = 0
     $txtScriptCode.Focus()
   } else {
     & $disableEdits
     $btnFixOffset.Enabled = $true
     $txtScriptCode.SelectionStart = 0
     $txtScriptCode.Focus()
     $lblStatus.Text="CarretOffset fixing is only enabled before editing!"
   }
   $txtTitle.Modified = $false
   $txtDescription.Modified = $false
   $txtScriptcode.Modified = $false 
   $form.gv_SnippetModified = $false
   & $txtScriptCode_TextChanged
   if($rdbFirstElipsis.Checked -eq $true) { & RadioButton_CheckedChanged($rdbFirstElipsis)}
   if($rdbFirstVar.Checked -eq $true) { & RadioButton_CheckedChanged($rdbFirstVar)}
   if($rdbCursorPos.Checked -eq $true) { & RadioButton_CheckedChanged($rdbCursorPos)}
   if($rdbCodeEnd.Checked -eq $true) { & RadioButton_CheckedChanged($rdbCodeEnd)} 
 }
 $nullEdits = {
   $txtTitle.Text=$null
   $txtDescription.Text=$null
   $txtScriptCode.Text=$null
   $txtTitle.Modified = $false
   $txtDescription.Modified = $false
   $txtScriptcode.Modified = $false 
 }
 $btnEditClick = {if($btnEdit.Text -eq "Edit") {
     $lblStatus.Text="Only the Title, Description and ScriptCode fields are enabled for edit!"
     & $enableEdits
     if (!($txtTitle.Text)) {
       $txtTitle.Focus()
     } else {
       $txtScriptCode.SelectionStart = $txtCaretOffset.Text
       $txtScriptCode.Focus()
     }
   } else {
     $lblStatus.Text="All fields are view only. (Editing is disabled)"
     & $disableEdits
   }
 }
 $enableEdits = {
   $btnFixOffset.Enabled = $false
   $txtTitle.ReadOnly=$false
   $txtDescription.ReadOnly=$false
   $txtScriptCode.ReadOnly=$false
   $txtTitle.Modified=$false
   $txtDescription.Modified=$false
   $txtScriptCode.Modified=$false
   $btnEdit.Text = "Lock"
 }
 $disableEdits = {
   $txtTitle.ReadOnly=$true
   $txtDescription.ReadOnly=$true
   $txtScriptCode.ReadOnly=$true
   $btnEdit.Text = "Edit"
 }
 $btnSaveClick = {
   $form.gv_SaveSnippetFlag = $true
   if (($txtFilePath.text.Length) -lt 10) {
     $lblStatus.Text="No Title or File Name provided for snippet!"
     $form.gv_SaveSnippetFlag = $false
   }
   if (($txtDescription.text.Length) -lt 10) {
     if ($form.gv_SaveSnippetFlag) {
       $lblStatus.Text="No snippet description provided yet, please add it!"
       $form.gv_SaveSnippetFlag = $false
     } else {
       $lblStatus.Text=($lblStatus.Text + "  No snippet description provided yet, please add it!")
     }
   }
   if (($txtScriptCode.text.Length) -lt 10) {
     if ($form.gv_SaveSnippetFlag) {
       $lblStatus.Text="No snippet script Code provided yet, please add it!"
       $form.gv_SaveSnippetFlag = $false
     } else {
       $lblStatus.Text=($lblStatus.Text + "  No snippet script Code provided yet, please add it!")
     }
   }
   if ($form.gv_SaveSnippetFlag -eq $true) {
     $oldFileName = ($txtFilePath.Text).Split("\")[-1]
     $msg = ("Confirm overwriting the existing file name:`r`n" + $oldFileName)
     $answ = [windows.forms.messagebox]::show($msg,"SAVE SNIPPET: Confirm Overwrite?",[system.windows.forms.messageboxbuttons]::yesno,[System.Windows.Forms.MessageBoxIcon]::Question)
     if ($answ -eq "Yes") {
       $form.gv_SaveSnippetFlag = $true
       $lblStatus.Text=("File is to be overwritten (add save file code)`r`nsaveFlag: " + $form.gv_SaveSnippetFlag) 
     } else {
       $form.gv_SaveSnippetFlag = $false
       $lblStatus.Text=("File Exists. Overwrite denied. Change the Title to create a new filename...`r`nsaveFlag: " + $form.gv_SaveSnippetFlag) 
       $txtTitle.Focus()
     }
   } else {
     $lblStatus.Text=("New file Editing is incomplete, the new filename is generated by the Title entry...") 
   }
   if ($form.gv_SaveSnippetFlag -eq $true) {& $saveSnippetXml}
 }
 $confirmFilenameChange = {
   $answ = $null
   $oldFileName = ($txtFilePath.Text).Split("\")[-1]        
   $tmp = $txtTitle.Text
   $t = '*,_,?,_,/,_,|,_,\,_,<,_,>,_,[,_,],_'
   $tmpFileName = Invoke-Expression ('$tmp' + -join $(foreach($e in $t.Split(',')) {'.Replace("{0}","{1}")' -f $e, $([void]$foreach.MoveNext();$foreach.Current)} ))
   $tmpFileName = ($tmpFileName + ".snippets.ps1xml")
   $msg = ("Confirm existing file name:`n" + $oldFileName + "`n`nIs to be replaced with new file name:`n" + $tmpFileName + "`n`nAnswering No changes the Title without changing the FileName!")
   if ($txtFilePath.Text) {
     $answ = [windows.forms.messagebox]::show($msg,"TITLE CHANGED: Confirm File Name Change?",[system.windows.forms.messageboxbuttons]::yesno,[System.Windows.Forms.MessageBoxIcon]::Question)
   } else {
     $answ = "Yes"
   }
   if ($answ -eq "Yes") {
     $txtFilePath.Text = Join-Path -Path $snippetPath -ChildPath $tmpFileName
     $md = "{0:y}" -f (Get-Date)
     $form.gv_FileLastWriteTime = $md
     & $UpdateDescription
   }
 }
 $txtTitle_LostFocus = {
   if ($txtTitle.Modified) {
     if (($txtTitle.Text)) {
       & $confirmFilenameChange
       $txtTitle.Modified
     } else {
       $txtTitle.Modified
       $lblStatus.Text = (($form.Tag) + "`nA Title is required as Filename is generated by the Title entry...")
     }
   }
 }
 
 $txtTitle_TextChanged = {
   if ($btnEdit.Text -eq "Lock") {
     $form.gv_SnippetModified = $true
   }
 }
 
 $txtDescription_TextChanged = {
   if ($btnEdit.Text -eq "Lock") {
     $form.gv_SnippetModified = $true
   }
 }
 
 $txtScriptCode_TextChanged = {
   if ($btnEdit.Text -eq "Lock") {$form.gv_SnippetModified = $true}
   $txtScriptCode.CodeEndPos = ($txtScriptCode.Text.Length)
   $txtCodeEndOffset.Text = (1 + ($txtScriptCode.CodeEndPos) - ($txtScriptCode.Lines.Count))
     $txtScriptCode.CursorLine = Get-CodeLine(($txtCursorOffset.Text))
     $tempOffset = (($txtScriptCode.CursorLine) + ($txtCursorOffset.Text))
     $txtScriptCode.CursorPos = ($tempOffset)
   } elseif ($txtScriptCode.SelectionStart -gt 0) {
     $txtScriptCode.CursorPos = $txtScriptCode.SelectionStart
     $txtScriptCode.CursorLine = Get-CodeLine($txtScriptCode.CursorPos)
     if ($txtScriptCode.CursorPos -gt 0) {
       $form.gv_FileCursorOffset = Get-CodeOffset($txtScriptCode.CursorPos)
     }
     $txtCursorOffset.Text = ($form.gv_FileCursorOffset)
   }
   & $getCaretOffset
   $lblStatus.Text = (($form.Tag) + "`nLast insertion point Cursor Offset position: " + ($form.gv_FileCursorOffset) + ", Code End insertion point will be " + ($txtCodeEndOffset.Text) +  ".")
 }
 
 $UpdateDescription = {
   $tmpDesc = $txtDescription.Text
   $sFlag = $tmpDesc.IndexOf([char]27)
   $lFlag = $tmpDesc.LastIndexOf([char]27)
   $eFlag = $tmpDesc.IndexOf([char]29)
   $patterns = @("\s*\;\s*Filename:\s","\s*Filename:\s","\s*\;\s*Modified:\s","\s*Modified:\s","\;\s+")
   for ($i=0;$i -lt $patterns.Count;$i++) {
     $pattern = $patterns[($i)]
     $f = $tmpDesc | Select-String -Pattern $pattern
     if ($f.Matches.Index -ge 0) {
       $fFlag = $f.Matches.Index
       $i = $patterns.Count
     }
   }
   $regex = "\x1B.*\x1D"
   if (($form.gv_FileLastWriteTime)) {
     $dts = "{0:MMM-dd-yyyy}" -f (get-date ($form.gv_FileLastWriteTime))
   } else {  
     $dts = "{0:MMM-dd-yyyy}" -f (get-date)
   }
   if ($c) {}
   $FileName = ($txtFilePath.Text).Split("\")[-1]
   if (!$FileName) {$FileName = "''"}
   $newd = ([char]27 + "; Filename: " + $FileName + "; Modified: " + $dts + [char]29)
     $s =  $tmpDesc.Substring(0,$sFlag)
     $s = ($s.trim() + $newd)
     $txtDescription.Text = $s
     if ($eFlag -ge 0) {
       if ($fFlag -ge 0) {
         $s =  $tmpDesc.Substring(0,$fFlag)
         $s = ($s.Trim() + $newd)
         $txtDescription.Text = $s
       } else {
         $s =  $tmpDesc.Replace([char]29,$null)
         $s = ($s.Trim() + $newd)
         $txtDescription.Text = $s
       }
       $s = ($tmpDesc.Trim() + $newd)
       $txtDescription.Text = $s
     }
   }
 }
 
 $saveSnippetXml = {
 $template = @"
 <?xml version='1.0' encoding='utf-8' ?>
     <Snippets  xmlns='http://schemas.microsoft.com/PowerShell/Snippets'>
         <Snippet Version='1.0.0'>
             <Header>
                 <Title></Title>
                 <Description></Description>
                 <Author>$form.gv_SnippetAuthor</Author>
                 <SnippetTypes>
                     <SnippetType>Expansion</SnippetType>
                 </SnippetTypes>
             </Header>
             <Code>
                 <Script Language='PowerShell' CaretOffset='1'>
                     <![CDATA[]]>
                 </Script>
             </Code>
     </Snippet>
 </Snippets>
 "@
   if (!(Test-Path ($txtFilePath.Text))) {
     Out-File -InputObject $template -FilePath ($txtFilePath.Text) -Encoding utf8 -NoClobber
   }
   $caretOffset = $txtCodeEndOffset.Text
   if (($rdbFirstElipsis.Checked)) {$caretOffset = $txtVarOffset.Text}
   if (($rdbFirstVar.Checked)) {$caretOffset = $txtVarOffset.Text}
   if (($rdbCursorPos.Checked)) {$caretOffset = $txtCursorOffset.Text}
   [xml]$snippetXml = Get-Content ($txtFilePath.Text)
   $snippetXml.Snippets.Snippet.Header.Title = $txtTitle.Text
   $snippetXml.Snippets.Snippet.Header.Description = $txtDescription.Text
   $snippetXml.Snippets.Snippet.Header.Author = $txtAuthor.Text
   $snippetXml.Snippets.Snippet.Header.SnippetTypes.SnippetType = $txtSnippetType.Text
   $snippetXml.Snippets.Snippet.Code.Script.SetAttribute('Language', $txtLanguage.Text)
   $snippetXml.Snippets.Snippet.Code.Script.SetAttribute('CaretOffset', $CaretOffset)
   [array]$codeLines = $txtScriptCode.Lines
   [string]$codeText = ""
   $max = ($codeLines.Count - 1)
   for ($i=0;$i -lt $max;$i++) {
     $codeText = ($codeText + $codeLines[($i)] + "`n")
   }
   $codeText = ($codeText + $codeLines[($i)])
   $snippetXml.Snippets.Snippet.Code.Script.FirstChild.InnerText = ($codeText)
   $snippetXml.Save(($txtFilePath.Text))
   $lblStatus.Text = ("Snippet: " + $txtTitle.Text + " saved.")
   Write-Warning $snippetXml.InnerXml
   $txtCaretOffset.text = $CaretOffset
   & $loadXml
 }
 
 $getCaretOffset = {
   $txtScriptCode.Elipsis1Pos = (1 + ($txtScriptCode.Text.IndexOf('{')))
   $form.gv_FileElipsisOffset = Get-CodeOffset($txtScriptCode.Elipsis1Pos)
   $lblVarOffset.Text = "1st Elipsis:"
   $txtVarOffset.Text = ($form.gv_FileElipsisOffset)
   $txtScriptCode.Variable1Pos = (1 + ($txtScriptCode.Text.IndexOf('$')))
   $form.gv_FileVariableOffset = Get-CodeOffset($txtScriptCode.Variable1Pos)
   if (($form.gv_FileVariableOffset) -gt 0) {$form.Tag = (($form.Tag) + "Variable found at position " + ($form.gv_FileVariableOffset) + ".")} else {$form.Tag = (($form.Tag) + "Variable not found in snippet code.")}
   if (!($rdbFirstElipsis.Checked)) {
     $lblVarOffset.Text = "1st Variable:"
     $txtVarOffset.Text = ($form.gv_FileVariableOffset)
   }
 }
 
 $fixCaretOffset = {
   $caretOffset = $txtCodeEndOffset.Text
   if (($rdbFirstElipsis.Checked)) {$caretOffset = $txtVarOffset.Text}
   if (($rdbFirstVar.Checked)) {$caretOffset = $txtVarOffset.Text}
   if (($rdbCursorPos.Checked)) {$caretOffset = $txtCursorOffset.Text}
   if (!($txtTitle.Modified) -and !($txtDescription.Modified) -and !($txtScriptCode.Modified) -and (($txtFilePath.Text) -ne "") ) {
     if (($txtFilePath.Text) -ne $null) {
       $msg = ("Confirm saving changes to ONLY the CaretOffset Field?`n`nNew CaretOffset value: $caretOffset`n`nNote: Offset based on your RadioButton selection below!")
       $answ = [windows.forms.messagebox]::show($msg,"CHANGE CARET OFFSET:",[system.windows.forms.messageboxbuttons]::yesno,[System.Windows.Forms.MessageBoxIcon]::Question)
       if ($answ -eq "Yes") {
         [xml]$snippetXml = Get-Content ($txtFilePath.Text)
         $oldOffset = $snippetXml.Snippets.Snippet.Code.Script.GetAttribute('CaretOffset')
         $snippetXml.Snippets.Snippet.Code.Script.SetAttribute('CaretOffset', $caretOffset)
         $snippetXml.Save(($txtFilePath.Text))
         $txtCaretOffset.Text = $caretOffset
         $lblStatus.Text=("Old CaretOffset value: " + $oldOffset + " replaced with current (" + ($txtCaretOffset.Text) + ")")
         & $enableEdits
       }
     }
   }
 }
 
 $AutoEditHelp = {$lblStatus.Text = "A check here Opens snippets directly in Edit mode (not locked), in the future."}
 
 $RemoveBlankLines = {
   $lines = @($txtScriptCode.Lines)
   $txtScriptCode.Text = $null
   for($i=0;$i -lt $lines.Count;$i++) {
     if ($lines[($i)] -notmatch '^\s*$') {
       $txtScriptCode.Text = ($txtScriptCode.Text + $lines[($i)] + "`r`n")
     }
   }
 }
 
 $NoBlanksHelp = {
   $lblStatus.Text = "A check here removes all blank lines from snippets opened in the future too."
   if ($ckbNoBlanks.Checked -eq $true) {& $RemoveBlankLines}
 }
 
 $lblFont = new-object System.Drawing.Font("Microsoft Sans Serif", 8.25, [System.Drawing.FontStyle]::Bold)
 
 $lblTitle.AutoSize = $true
 $lblTitle.Font = $lblFont
 $lblTitle.Location = new-object System.Drawing.Point(35, 9)
 $lblTitle.Name = "lblTitle"
 $lblTitle.Size = new-object System.Drawing.Size(32, 13)
 $lblTitle.Text = "Title:"
 $form.Controls.Add($lblTitle)
             
 $txtTitle.Location = new-object System.Drawing.Point(12, 27)
 $txtTitle.Name = "txtTitle"
 $txtTitle.ReadOnly = $true
 $txtTitle.Size = new-object System.Drawing.Size(473, 20)
 $txtTitle.TabIndex = 0 
 $txtTitle.Add_LostFocus($txtTitle_LostFocus)
 $txtTitle.Add_TextChanged($txtTitle_TextChanged)
 $form.Controls.Add($txtTitle)
 
 $lblAuthor.AutoSize = $true
 $lblAuthor.Font = $lblFont
 $lblAuthor.Location = new-object System.Drawing.Point(567, 9)
 $lblAuthor.Name = "lblAuthor"
 $lblAuthor.Size = new-object System.Drawing.Size(44, 13)
 $lblAuthor.Text = "Author:"
 $lblAuthor.Add_MouseDoubleClick($ChangeDefaultAuthor)
 $lblAuthor.Add_Click({$lblStatus.Text = ("A DoubleClick in the Author Label will allow you to enter your name...`nA DoubleClick on the Author TextBox below it will replace the current Author with " + ($form.gv_SnippetAuthor))})
 $form.Controls.Add($lblAuthor)
             
 $txtAuthor.Location = new-object System.Drawing.Point(555, 27)
 $txtAuthor.Name = "txtAuthor"
 $txtAuthor.ReadOnly = $true
 $txtAuthor.AcceptsReturn = $false
 $txtAuthor.AcceptsTab = $false
 $txtAuthor.Multiline = $false
 $txtAuthor.Size = new-object System.Drawing.Size(145, 20)
 $txtAuthor.Text = $form.gv_SnippetAuthor
 $txtAuthor.TabStop = $false
 $txtAuthor.Add_MouseDoubleClick({$this.Text = ($form.gv_SnippetAuthor)})
 $txtAuthor.Add_Click({$lblStatus.Text = ("A DoubleClick in the Author TextBox will replace the current Author with " + ($form.gv_SnippetAuthor) + "`nA DoubleClick on the Author Label above it will allow you to enter your name...")})
 Add-Member -InputObject $txtAuthor -MemberType NoteProperty -Name TempAuthor -Value "" -TypeName [string]
 $form.Controls.Add($txtAuthor)
             
 $lblDescription.AutoSize = $true
 $lblDescription.Font = $lblFont
 $lblDescription.Location = new-object System.Drawing.Point(35, 58)
 $lblDescription.Name = "lblDescription"
 $lblDescription.Size = new-object System.Drawing.Size(71, 13)
 $lblDescription.Text = "Description:"
 $form.Controls.Add($lblDescription)
             
 $txtDescription.Location = new-object System.Drawing.Point(12, 76)
 $txtDescription.Name = "txtDescription"
 $txtDescription.ReadOnly = $true
 $txtDescription.Size = new-object System.Drawing.Size(886, 20)
 $txtDescription.TabIndex = 1
 $txtDescription.Add_TextChanged($txtDescription_TextChanged)
 $form.Controls.Add($txtDescription)
             
 $lblExpansion.Font = $lblFont
 $lblExpansion.Location = new-object System.Drawing.Point(765, 9)
 $lblExpansion.Name = "lblExpansion"
 $lblExpansion.Size = new-object System.Drawing.Size(82, 13)
 $lblExpansion.Text = "Snippet Type:"
 $form.Controls.Add($lblExpansion)
             
 $txtSnippetType.Location = new-object System.Drawing.Point(753, 27)
 $txtSnippetType.Name = "txtSnippetType"
 $txtSnippetType.ReadOnly = $true
 $txtSnippetType.Size = new-object System.Drawing.Size(145, 20)
 $txtSnippetType.Text = "Expansion"
 $txtSnippetType.TabStop = $false
 $form.Controls.Add($txtSnippetType)
             
 $lblLanguage.Font = $lblFont
 $lblLanguage.Location = new-object System.Drawing.Point(230, 119)
 $lblLanguage.Name = "lblLanguage"
 $lblLanguage.Size = new-object System.Drawing.Size(63, 13)
 $lblLanguage.Text = "Language:"
 #$lblLanguage.Click += new-object System.EventHandler($label2_Click_1)
 $form.Controls.Add($lblLanguage)
             
 $txtLanguage.Location = new-object System.Drawing.Point(300, 116)
 $txtLanguage.Name = "txtLanguage"
 $txtLanguage.ReadOnly = $true
 $txtLanguage.Size = new-object System.Drawing.Size(145, 20)
 $txtLanguage.Text = "Powershell"
 $txtLanguage.TabStop = $false
 $form.Controls.Add($txtLanguage)
             
 $lblCursorOffset.Font = $lblFont
 $lblCursorOffset.Location = new-object System.Drawing.Point(600, 119)
 $lblCursorOffset.Name = "lblCursorOffset"
 $lblCursorOffset.Size = new-object System.Drawing.Size(80, 13)
 $lblCursorOffset.Text = "Cursor Offset:"
 $form.Controls.Add($lblCursorOffset)
             
 $txtCursorOffset.Location = new-object System.Drawing.Point(685, 116)
 $txtCursorOffset.Name = "txtCursorOffset"
 $txtCursorOffset.ReadOnly = $true
 $txtCursorOffset.Size = new-object System.Drawing.Size(55, 20)
 $txtCursorOffset.Text = "0"
 $txtCursorOffset.TabStop = $false
 $form.Controls.Add($txtCursorOffset)
 
 $lblVarOffset.Font = $lblFont
 $lblVarOffset.Location = new-object System.Drawing.Point(600, 100)
 $lblVarOffset.Name = "lblVarOffset"
 $lblVarOffset.Size = new-object System.Drawing.Size(75, 13)
 $lblVarOffset.Text = "1st Variable:"
 $form.Controls.Add($lblVarOffset)
 
 $txtVarOffset.Location = new-object System.Drawing.Point(685, 97)
 $txtVarOffset.Name = "txtVarOffset"
 $txtVarOffset.ReadOnly = $true
 $txtVarOffset.Size = new-object System.Drawing.Size(55, 20)
 $txtVarOffset.Text = "0"
 $txtVarOffset.TabStop = $false
 $form.Controls.Add($txtVarOffset)
    
 $lblCaretOffset.Font = $lblFont
 $lblCaretOffset.Location = new-object System.Drawing.Point(762, 119)
 $lblCaretOffset.Name = "lblCaretOffset"
 $lblCaretOffset.Size = new-object System.Drawing.Size(75, 13)
 $lblCaretOffset.Text = "Caret Offset:"
 $lblCaretOffset.Add_Click($fixCaretOffset)
 $form.Controls.Add($lblCaretOffset)
             
 $txtCaretOffset.Location = new-object System.Drawing.Point(843, 116)
 $txtCaretOffset.Name = "txtCaretOffset"
 $txtCaretOffset.ReadOnly = $true
 $txtCaretOffset.Size = new-object System.Drawing.Size(55, 20)
 $txtCaretOffset.Text = "1"
 $txtCaretOffset.TabStop = $false
 $txtCaretOffset.Add_Click($fixCaretOffset)
 $form.Controls.Add($txtCaretOffset)
 
 $lblCodeEndOffset.Font = $lblFont
 $lblCodeEndOffset.Location = new-object System.Drawing.Point(761, 100)
 $lblCodeEndOffset.Name = "lblCodeEndOffset"
 $lblCodeEndOffset.Size = new-object System.Drawing.Size(75, 13)
 $lblCodeEndOffset.Text = "Code End:"
 $form.Controls.Add($lblCodeEndOffset)
 
 $txtCodeEndOffset.Location = new-object System.Drawing.Point(843, 97)
 $txtCodeEndOffset.Name = "txtCodeEndOffset"
 $txtCodeEndOffset.ReadOnly = $true
 $txtCodeEndOffset.Size = new-object System.Drawing.Size(55, 20)
 $txtCodeEndOffset.Text = "1"
 $txtCodeEndOffset.TabStop = $false
 $form.Controls.Add($txtCodeEndOffset)
 
 $lblScriptCode.Font = $lblFont
 $lblScriptCode.Location = new-object System.Drawing.Point(35, 116)
 $lblScriptCode.Name = "lblScriptCode"
 $lblScriptCode.Size = new-object System.Drawing.Size(73, 13)
 $lblScriptCode.Text = "Script Code:"
 $form.Controls.Add($lblScriptCode)
             
 $txtScriptCode.AcceptsReturn = $true
 $txtScriptCode.AcceptsTab = $true
 $txtScriptCode.AllowDrop = $true
 $txtScriptCode.Location = new-object System.Drawing.Point(12, 142)
 $txtScriptCode.Multiline = $true
 $txtScriptCode.ScrollBars = [System.Windows.Forms.ScrollBars]::Both
 $txtScriptCode.Name = "txtScriptCode"
 $txtScriptCode.ReadOnly = $true
 $txtScriptCode.Size = new-object System.Drawing.Size(886, 294)
 $txtScriptCode.TabIndex = 2
 $txtScriptCode.WordWrap = $false
 $txtScriptCode.Add_TextChanged($txtScriptCode_TextChanged)
 Add-Member -InputObject $txtScriptCode -MemberType NoteProperty -Name Elipsis1Pos -Value 0
 Add-Member -InputObject $txtScriptCode -MemberType NoteProperty -Name Variable1Pos -Value 0
 Add-Member -InputObject $txtScriptCode -MemberType NoteProperty -Name CursorPos -Value 0
 Add-Member -InputObject $txtScriptCode -MemberType NoteProperty -Name CursorLine -Value 0
 Add-Member -InputObject $txtScriptCode -MemberType NoteProperty -Name CodeEndPos -Value 1
 $form.Controls.Add($txtScriptCode)
 
 $lblFilePath.Font = $lblFont
 $lblFilePath.Location = new-object System.Drawing.Point(35, 455)
 $lblFilePath.Name = "lblFilePath"
 $lblFilePath.Size = new-object System.Drawing.Size(65, 13)
 $lblFilePath.Text = "File Path:"
 $form.Controls.Add($lblFilePath)
             
 $txtFilePath.Location = new-object System.Drawing.Point(105, 450)
 $txtFilePath.Name = "txtFilePath"
 $txtFilePath.ReadOnly = $true
 $txtFilePath.Size = new-object System.Drawing.Size(791, 20)
 $txtFilePath.Text = ""
 $form.Controls.Add($txtFilePath)
             
 $lblStatus.Location = new-object System.Drawing.Point(12, 505)
 $lblStatus.Name = "lblStatus"
 $lblStatus.Size = new-object System.Drawing.Size(886, 30)
 $lblStatus.Text = ""
 $lblStatus.TextAlign = [System.Drawing.ContentAlignment]::MiddleCenter
 $form.Controls.Add($lblStatus)
 
 $btnPsIse.Location = new-object System.Drawing.Point(25, 480)
 $btnPsIse.Name = "btnPsIse"
 $btnPsIse.Size = new-object System.Drawing.Size(75, 23)
 $btnPsIse.TabIndex = 3
 $btnPsIse.Text = "In PsIse"
 $btnPsIse.UseVisualStyleBackColor = $true
 $btnPsIse.add_click($btnPsIseClick)
 $form.Controls.Add($btnPsIse)
 
 $btnOpen.Location = new-object System.Drawing.Point(125, 480)
 $btnOpen.Name = "btnOpen"
 $btnOpen.Size = new-object System.Drawing.Size(75, 23)
 $btnOpen.TabIndex = 4
 $btnOpen.Text = "Open"
 $btnOpen.UseVisualStyleBackColor = $true
 $btnOpen.add_click($btnOpenClick)
 $form.Controls.Add($btnOpen)
 
 $ckbAutoEdit.Location = new-object System.Drawing.Point(215, 472)
 $ckbAutoEdit.Name = "ckbAutoEdit"
 $ckbAutoEdit.TabIndex = 5
 $ckbAutoEdit.Text = "Auto Edit"
 $ckbAutoEdit.AutoSize = $true
 $ckbAutoEdit.Add_Click($AutoEditHelp)
 $form.Controls.Add($ckbAutoEdit)
 
 $ckbNoBlanks.Location = new-object System.Drawing.Point(215, 490)
 $ckbNoBlanks.Name = "ckbNoBlanks"
 $ckbNoBlanks.TabIndex = 5
 $ckbNoBlanks.Text = "No Blanks"
 $ckbNoBlanks.AutoSize = $true
 $ckbNoBlanks.Add_Click($NoBlanksHelp)
 $form.Controls.Add($ckbNoBlanks)
 
 $btnNew.Location = new-object System.Drawing.Point(300, 480)
 $btnNew.Name = "btnNew"
 $btnNew.Size = new-object System.Drawing.Size(75, 23)
 $btnNew.TabIndex = 6
 $btnNew.Text = "New"
 $btnNew.UseVisualStyleBackColor = $true
 $btnNew.Add_click($do_btnNew)
 $form.Controls.Add($btnNew)
             
 $btnEdit.Location = new-object System.Drawing.Point(400, 480)
 $btnEdit.Name = "btnEdit"
 $btnEdit.Size = new-object System.Drawing.Size(75, 23)
 $btnEdit.TabIndex = 7
 $btnEdit.Text = "Edit"
 $btnEdit.UseVisualStyleBackColor = $true
 $btnEdit.add_click($btnEditClick)
 $form.Controls.Add($btnEdit)
 
 $btnFixOffset.Location = new-object System.Drawing.Point(500, 480)
 $btnFixOffset.Name = "btnFixOffset"
 $btnFixOffset.Size = new-object System.Drawing.Size(75, 23)
 $btnFixOffset.TabIndex = 8
 $btnFixOffset.Text = "Fix Offset"
 $btnFixOffset.UseVisualStyleBackColor = $true
 $btnFixOffset.Enabled = $false
 $btnFixOffset.add_click($fixCaretOffset)
 $form.Controls.Add($btnFixOffset)
 
 $rdbFirstElipsis.Location = new-object System.Drawing.Point(600, 472)
 $rdbFirstElipsis.Name = "rdbFirstElipsis"
 $rdbFirstElipsis.TabIndex = 10
 $rdbFirstElipsis.Text = "1st Elipsis"
 $rdbFirstElipsis.AutoSize = $true
 $rdbFirstElipsis.Add_Click({RadioButton_CheckedChanged($this)})
 $form.Controls.Add($rdbFirstElipsis)
 
 $rdbFirstVar.Location = new-object System.Drawing.Point(600, 490)
 $rdbFirstVar.Name = "rdbFirstVar"
 $rdbFirstVar.TabIndex = 9
 $rdbFirstVar.Text = "1st Variable"
 $rdbFirstVar.AutoSize = $true
 $rdbFirstVar.Add_Click({RadioButton_CheckedChanged($this)})
 $form.Controls.Add($rdbFirstVar)
 
 $rdbCursorPos.Location = new-object System.Drawing.Point(685, 472)
 $rdbCursorPos.Name = "rdbCursorPos"
 $rdbCursorPos.TabIndex = 10
 $rdbCursorPos.Text = "Cursor Pos"
 $rdbCursorPos.AutoSize = $true
 $rdbCursorPos.Checked = $true
 $rdbCursorPos.Add_Click({RadioButton_CheckedChanged($this)})
 #$rdbCursorPos.Add_CheckedChanged({RadioButton_CheckedChanged($this)})
 $form.Controls.Add($rdbCursorPos)
 
 $rdbCodeEnd.Location = new-object System.Drawing.Point(685, 490)
 $rdbCodeEnd.Name = "rdbCodeEnd"
 $rdbCodeEnd.TabIndex = 10
 $rdbCodeEnd.Text = "Code End"
 $rdbCodeEnd.AutoSize = $true
 $rdbCodeEnd.Add_Click({RadioButton_CheckedChanged($this)})
 $form.Controls.Add($rdbCodeEnd)
 
 $btnSave.Location = new-object System.Drawing.Point(800, 480)
 $btnSave.Name = "btnSave"
 $btnSave.Size = new-object System.Drawing.Size(75, 23)
 $btnSave.TabIndex = 11
 $btnSave.Text = "Save"
 $btnSave.UseVisualStyleBackColor = $true
 $btnSave.add_click($btnSaveClick)
 $form.Controls.Add($btnSave)
 
 $form.Add_Closing({Check_UnsavedSnippet($form)})
 $form.Add_KeyDown({if ($_.KeyCode -eq "Enter") {if (($txtAuthor.TabStop)) {& $ValidateAuthor}}})
 $form.Add_KeyDown({if ($_.KeyCode -eq "Escape") {if (($txtAuthor.TabStop)) {$txtAuthor.Text = $null; & $ValidateAuthor}}})
 
 $form.ShowDialog() | Out-Null
`

