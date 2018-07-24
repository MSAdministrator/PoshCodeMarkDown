---
Author: bartek bielawski (@bielawb on twitter)
Publisher: 
Copyright: 
Email: 
Version: 0.2
Encoding: ascii
License: cc0
PoshCode ID: 2490
Published Date: 
Archived Date: 2011-02-09t06
---

# isefun.psm1 - 

## Description

adds add-ons menu 'isefun' with all functions included.

## Comments



## Usage



## TODO



## module

`add-mymenuitem`

## Code

`#
 #
     
 
 
 
 if (-not ($MyISEMenu = $psISE.CurrentPowerShellTab.AddOnsMenu.Submenus | Where-Object { $_.DisplayName -eq 'ISEFun'} ) ) {
     $MyISEMenu = $psISE.CurrentPowerShellTab.AddOnsMenu.Submenus.Add('ISEFun',$null,$null)
 }
 
 function Add-MyMenuItem {
 
 <#
     .Synopsis
         Adds items to ISEFun Add-Ons sub-menu
     .Description
         Function can be used to add menu items to ISEFun menu. All you need is command, name and hotkey - we will take care of the rest for you. ;)
     .Example
         Add-MyMenuItem 'Write-Host fooo' 'Fooo!' 'CTRL+9'
         
         Description
         -----------
         This command will add item 'Fooo!' to ISEFun menu. This item will write 'fooo' to the host and can be launched using shortcut CTRL + 9
         
 #>
 
     PARAM (
         [Parameter(Mandatory = $true, HelpMessage = 'Command that you want to add to menu')]
         [string]$Command,
         [string]$DisplayName,
         [string]$HotKey = $null,
         [string]$SubMenu = $null
     )
     if (!$DisplayName) { 
         $DisplayName = $Command -replace '-',' '
     }
     if (!$SubMenu) {
         $Menu = $Script:MyISEMenu
     } elseif ($Menu = $MyISEMenu.Submenus | Where-Object { $_.DisplayName -eq $SubMenu }) {
         if (!($Menu.submenus)) {
             $Menu = $Script:MyISEMenu
         }
     } else {
         $Menu = $MyISEMenu.Submenus.Add($SubMenu,$null,$null)
     }
     if ( -not ($Menu.Submenus | Where-Object { $_.DisplayName -eq $DisplayName} ) ) {
         try {
             [void]$Menu.Submenus.Add($DisplayName,[scriptblock]::Create($Command),$HotKey)
         } catch {
             [void]$Menu.Submenus.Add($DisplayName,[scriptblock]::Create($Command),$null)
         }
     }
 }
 
 Add-MyMenuItem Add-MyMenuItem 'Add items'
 
 function Update-SnippetMenu {
 
 <#
     .Synopsis
         Updates/ creates custom snippet menu.
     .Description
         This function will take any snippets that you have in location (either default or selected one) and add those to ISEFun 'Code Snippets' menu.
         It uses .ps1 file to make it easier to modify files in ISE (with syntax highlighting and stuff...)
     .Example
         Update-SnippetMenu
         Checks for files in $PSScriptRoot\ISESnippets folder and add them to 'Code Snippets' menu.
 #>
     param (
         [string]$Path = "$PSScriptRoot\ISESnippets"
         )
     foreach ($file in Get-ChildItem $Path -filter *.ps1) {
         $command = @"
         if (!(`$file = `$psISE.CurrentFile)) {
             `$file = `$psISE.CurrentPowerShellTab.Files.Add()
         }
         `$file.Editor.InsertText(
           `$( (Get-Content $($file.FullName))  | Out-String )
           )
 "@
         $Name = $file.BaseName -replace '_', ' '
         Add-MyMenuItem $command "Instert $Name" $null 'Code Snippets'
     }
 }
 
 Add-MyMenuItem Update-SnippetMenu 'Adds/ updates code snippets menu'
 
 function New-Snippet {
 
 <#
     .Synopsis
         Creates new snippet from current file/ selection.
     .Description
         This function will create new snippets for you. To simplify usage in ISE it has mandatory parameter for snippet name (GUI prompt).
         It will by default store snippets under modules root folder in 'ISESnippets'.
     .Example
         New-Snippet -SnippetName 'My funny name that will have underscore instead of **?? special \ chars'
         Will create new file (with rather odd name) with current selection (or file, if nothing was selected).
         All special chars and spaces will be replace with underscore ( _ ).
 #>
     param(
         [Parameter(Mandatory = $true, HelpMessage = 'Please specify name of the snippet')]
         [Alias("SN","Name")]
         [string]$SnippetName,
         [string]$Path = "$PSScriptRoot\ISESnippets"
     )
     if ($file = $psISE.CurrentFile) {
         if (!($Value = $file.Editor.SelectedText)) {
             $Value = $file.Editor.Text
         }   
     } else {
         Write-Error "No files opened in this tab!"
         return
     }
     $Name = ($SnippetName -replace '[\s+\\\*\?]', '_') + '.ps1'
     New-Item -Force -ItemType file -Path (Join-Path $Path $Name) -Value $Value | Out-Null
 }
 
 Add-MyMenuItem New-Snippet 'Create new snippet from selection/ file'
 
 function Show-Top {
     PARAM (
         [int]$Count = 20,
         [int]$Sleep = 5,
         [string]$Property = 'WS'
         )
     $TopTab = $psISE.PowerShellTabs.Add()
     sleep 10
     if ($TopTab.CanInvoke) {
         $TopTab.DisplayName = "TOP$Count - $Property"
         $psISE.PowerShellTabs.SetSelectedPowerShellTab($TopTab)
         $TopTab.Invoke([scriptblock]::Create(@"
             Register-WmiEvent -Query "SELECT * From Win32_ProcessStartTrace"
             Register-WmiEvent -Query "SELECT * From Win32_ProcessStopTrace"
             `$formaterS = "{0,10} {1,10} {2,10} {3,10} {4,10} {5,10} {6,10} {7,-30}"
             `$formaterD = "{0,10} {1,10:N0} {2,10:N0} {3,10:N0} {4,10:N0} {5,10:N2} {6,10} {7,-30}"
             while (`$true) { 
                 function prompt {
                     `$psISE.PowerShellTabs.Remove(`$psISE.CurrentPowerShellTab)
                 }
 
                 Clear-Host
                 'TOP for PowerShell ISE - from ISEFun module'
                 '=' * 50
                 `$Processes = Get-Process
                 
                 `$Started = @(Get-Event | where { `$_.SourceEventArgs.NewEvent.__Class -match 'Start' } | foreach { `$_.SourceEventArgs.NewEvent.ProcessID })
                 `$Stopped = @(Get-Event | where { `$_.SourceEventArgs.NewEvent.__Class -match 'Stop' } | foreach { `$_.SourceEventArgs.NewEvent.ProcessID })
                 `$Processes += `$DisplayedProcess | where { `$Stopped -contains `$_.Id }
                 `$Processes = `$Processes | sort -Descending $Property
                 `$DisplayedProcess = `$Processes | select -First $Count
                 `$formaterS -f `$('Handles NPM(K) PM(K) WS(K) VM(M) CPU(s) Id ProcessName'.Split())
                 `$formaterS -f `$(@('-' * 10) * 8)
                 foreach (`$process in `$DisplayedProcess) {
                     `$ProcessString = `$formaterD -f `$process.Handles, `$(`$process.NPM / 1KB), `$(`$process.PM/1KB), `$(`$process.WS / 1KB), `$(`$process.VM / 1MB), `$process.CPU, `$process.Id, `$(`$process.Name -replace '(^.{0,29}).*', '`$1')
                     if (`$Stopped -contains `$process.id) {
                         `$ProcessString + "==> PROCESS TERMINATED"
                     } elseif (`$Started -contains `$process.id) {
                         `$ProcessString + "<== NEW PROCESS ADDED"
                     } else {
                         `$ProcessString
                     }
                 }
                                                                                                                        
                 Get-Event | Remove-Event
                 
                 "Diplay $Count processes out of `$(`$Processes.Count)"
                 sleep $Sleep
             }
 "@))
     }
 }
 
 Add-MyMenuItem Show-Top
 
 function Edit-HelpExample {
 
 <#
     .Synopsis
         Simple function to copy help examples (as useable code) and it's description (as comments) to empty file in ISE
     .Description
         If you want to see what examples are available for a given command - that function is for you.
         It can easily get all examples in a way that you can just highlight them and get results back.
         Only basic testing done, so probably something is still missing...
     .Example
         Get-HelpExample Get-ChildItem
         Copies all 7 examples from Get-ChilItem cmdlet to empty file.
     .Example
         Get-HelpExample Get-HelpExample
         If I did everything right you will probably see this command, text that I'm currently typing and anything above/ below in this section of function
         
 #> 
  
     param (
     [Parameter(Mandatory = $true, HelpMessage = 'Name of the command that will be used to get examples')]
     [ValidateScript( { (Get-Help $_).Examples.example })]
     [string]$Name
     )
     if ($psISE) {
         $Editor = $psISE.CurrentPowerShellTab.Files.Add().Editor
         foreach ($example in (Get-Help $Name).Examples.example) {
                 $Editor.InsertText(@"
 
 $(($example.remarks | select -ExpandProperty text | where { $_ }) -join "`n")
 
 END :=> $(($example.title) -replace '-') #>
 
 $(($example | select -ExpandProperty code) -replace '(?m:^C:(\\\w*)+>)')
 
 "@)                
         }
     } else {
         throw "You must use this function/ script in PowerShell ISE!"
     }
 }
 
 Add-MyMenuItem Edit-HelpExample 'Edit Help Example'
 
 #
 $handler_bClose_Click= 
 {
     $Main.Hide()
 }
 
 $handler_bColor_Click= 
 {
     $Dialog = New-Object Windows.Forms.ColorDialog -Property @{
         Color = [drawing.color]::FromArgb($psISE.Options.TokenColors.Item($Combo.SelectedItem).ToString())
         FullOpen = $true
     }
     
     if ($Dialog.ShowDialog() -eq 'OK') {
         $psISE.Options.TokenColors.Item($Combo.SelectedItem) = [windows.media.color]::FromRgb($Dialog.Color.R, $Dialog.Color.G, $Dialog.Color.B)
         $Combo.ForeColor = $Dialog.Color
 
     }
 
 }
 
 $handler_selectedValue = {
     $Combo.ForeColor = [drawing.color]::FromArgb($psISE.Options.TokenColors.Item($Combo.SelectedItem).ToString())
     $bColor.Focus()
 }
 
 $OnLoadForm_StateCorrection = {
 	$Main.WindowState = $InitialFormWindowState
 }
 
 $Script:Main = New-Object Windows.Forms.Form -Property @{
     Text = "Token colors selector"
     MaximizeBox = $False
     Name = "Main"
     HelpButton = $True
     MinimizeBox = $False
     ClientSize = New-Object System.Drawing.Size 426, 36
 }
 $Main.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $Combo = New-Object Windows.Forms.ComboBox -Property @{
     FormattingEnabled = $True
     Size = New-Object System.Drawing.Size 239, 23
     Name = "Combo"
     Location = New-Object System.Drawing.Point 12, 7
     Font = New-Object System.Drawing.Font("Lucida Console",11.25,0,3,238)
     TabIndex = 4
     DropDownStyle = 'DropDownList'
     }
 $Combo.DataBindings.DefaultDataSourceUpdateMode = 0
 $Combo.Items.AddRange($psISE.Options.TokenColors.Keys)
 $Combo.Add_SelectedValueChanged($handler_SelectedValue)
 
 $InitialFormWindowState = New-Object Windows.Forms.FormWindowState
 
 $bClose = New-Object Windows.Forms.Button -Property @{
     TabIndex = 2 
     Name = "bClose"
     Size = New-Object System.Drawing.Size 75, 23
     UseVisualStyleBackColor = $True
     Text = "Close"
     Location = New-Object System.Drawing.Point 338, 7
     }
 $bClose.DataBindings.DefaultDataSourceUpdateMode = 0
 $bClose.add_Click($handler_bClose_Click)
 
 $bColor = New-Object Windows.Forms.Button -Property @{
     TabIndex = 1
     Name = "bColor"
     Size = New-Object System.Drawing.Size 75, 23
     UseVisualStyleBackColor = $True
     Text = "Color"
     Location = New-Object System.Drawing.Point 257, 7
 }
 $bColor.DataBindings.DefaultDataSourceUpdateMode = 0
 $bColor.add_Click($handler_bColor_Click)
 
 $Main.Controls.AddRange(@($bColor,$bClose,$Combo))
 $InitialFormWindowState = $Main.WindowState
 $Main.add_Load($OnLoadForm_StateCorrection)
 $HelpMessage = @'
 This GUI will help you change you token colors.
 It's updating text color as you select tokens that you want to modify.
 Button 'Color' opens up color dialog.
 I won't describe actions performed by 'Close' button. I hope you are able to guess it... ;)
 '@
 $Main.add_HelpButtonClicked( { [void][windows.forms.MessageBox]::Show($HelpMessage,'Help','OK','Information')})
 
 function Set-TokenColor {
 
 <#
     .Synopsis
         GUI to add some Token Colors.
     .Description
         Really. It is just that. No more to it. Seriously!
         OK. GUI is pretty smart. You can select tokens that are available, color will change and match the one you currently have. See for yourself. ;)
     .Example
         Can't show you click-click-click example :)
 #>
     $Script:Main.ShowDialog()| Out-Null
 }
 
 Add-MyMenuItem Set-TokenColor
 
 function Expand-Alias {
 
 <#
     .Synopsis
         Function to expand all command aliases in current script.
     .Description
         If you want to expand all aliases in a script/ module that you write in PowerShell ISE - this function will help you with that.
         It's using Tokenizer to find all commands, Get-Alias to find aliases and their definition, and simply replace alias with command hidden by it.
     .Example
         Expand-Alias
 #>
 
     if (!$psISE.CurrentFile) {
         throw 'No files opened!'
     }
     
     if ( -not ($Script = $psISE.CurrentFile.Editor.Text) ) {
         throw 'No code!'
     }
     
     $line = $psISE.CurrentFile.Editor.CaretLine
     $column = $psISE.CurrentFile.Editor.CaretColumn
     
     if ( -not ($commands = [System.Management.Automation.PsParser]::Tokenize($Script, [ref]$null) | Where-Object { $_.Type -eq 'Command' } | Sort-Object -Property Start -Descending) ) {
         return
     } 
     foreach ($command in $commands) {
         if (Get-Alias $command.Content -ErrorAction SilentlyContinue) {
             $psISE.CurrentFile.Editor.Select($command.StartLine, $command.StartColumn, $command.EndLine, $command.EndColumn)
             $psISE.CurrentFile.Editor.InsertText($(Get-Alias $command.Content | Select-Object -ExpandProperty Definition))
         }
     }
     $psISE.CurrentFile.Editor.SetCaretPosition($line, $column)
 }
 
 Add-MyMenuItem Expand-Alias
 
 function Edit-Function {
 
 <#
     .Synopsis
         Simpe function to edit functions in ISE.
     .Description
         Need to edit function on-the-fly? Want to see how a given function looks like to change it a bit and rename it?
         Or maybe just preparing module and you want to change functions you define to make sure changes will work as expected?
         Well, with Edit-Function, which is very simple (thank you PowerShell team!) you can do it. :)
     .Example
         Edit-Function Edit-Function
         
         Description
         -----------
         You can open any function that exists in your current session, including the function that you reading help to now.
         Be careful with that one though. If you change it in wrong direction you may not be able to open it again and fix it.
         At least not in the way you could originaly, with Edit-Function. :)
 #>
 
     [CmdletBinding()]
     param (
     [Parameter(Mandatory=$true,HelpMessage='Function name is mandatory parameter.')]
     [ValidateScript({Get-Command -CommandType function $_})]
     [string]
     $Name
     )
     if (!$psISE) {
         Throw 'Implemented for PowerShell ISE only!'
     }
     $file = $psISE.CurrentPowerShellTab.Files.Add()
     $file.Editor.InsertText("function $name {`n")
     $file.Editor.InsertText($(Get-Command -CommandType function $name | Select-Object -ExpandProperty definition))
     $file.Editor.InsertText("}")
 }
 
 Add-MyMenuItem Edit-Function
 
 function Copy-HistoryItem {
 
 <#
     .Synopsis
         Function build using WPK to give you functionality similar to one you already have in PowerShell.exe
     .Description
         Display you command history and let you choose from it. Copies selected command to you commandPane.
     .Example
         Copy-HistoryItem
         GUI, so it's not easy to show examples...
 #>
 
     try {
         New-Window -Width 800 -Height 100 {
             New-ListBox -On_PreviewMouseDoubleClick {
                 $psISE.CurrentPowerShellTab.CommandPane.InsertText($this.SelectedValue)
                 $this.parent.close()
             } -Items $(Get-History | Select-Object -ExpandProperty CommandLine)
         } -Show
     } catch {
         throw 'Requires WPK to work, will be rewritten soon...'
     }
 }
 
 Add-MyMenuItem Copy-HistoryItem 'Copy item from History' F7
 
 function Invoke-CurrentLine {
     $Editor = $psISE.CurrentFile.Editor
     $row = $Editor.CaretLine
     $col = $Editor.CaretColumn
     $Editor.Select($row, 1, $col, $Editor.GetLineLength($row) + 1)
     [scriptblock]::Create($Editor.SelectedText).Invoke()
     $Editor.SetCaretPosition($row,$col)
 }
 
 Add-MyMenuItem Invoke-CurrentLine 'Invoke Current Line' F9
 
 New-Alias -Name edfun -Value Edit-Function
 New-Alias -Name expa -Value Expand-Alias
 New-Alias -Name cphi -Value Copy-HistoryItem
 
 Export-ModuleMember -Function * -Alias *
 
 $MyInvocation.MyCommand.ScriptBlock.Module.OnRemove = {
     [void]$psISE.CurrentPowerShellTab.AddOnsMenu.Submenus.Remove($MyISEMenu)
 }
`

