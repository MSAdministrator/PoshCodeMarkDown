---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2801
Published Date: 2011-07-18t13
Archived Date: 2016-11-02t06
---

# new-isemenu - 

## Description

this is a little tweak of ravikanth’s script to generate a dialog for adding menu items in powershell ise.  all i did was just clean it up a little bit following a few guidelines

## Comments



## Usage



## TODO



## module

`new-isemenu`

## Code

`#
 #
 Import-Module ShowUI
 Function New-ISEMenu {
    New-Grid -AllowDrop:$true -Name "ISEAddonCreator" -columns Auto, * -rows Auto,Auto,Auto,*,Auto,Auto -Margin 5 {
       New-Label -Name Warning -Foreground Red -FontWeight Bold -Column 1
       ($target = New-TextBox -Name txtName -Column 1 -Row ($Row=1))
       New-Label "Addon Menu _Name" -Target $target  -Row $Row
       ($target = New-TextBox -Name txtShortcut -Column 1 -Row ($Row+=1))
       New-Label "Shortcut _Key" -Row $Row -Target $target
       ($target = New-TextBox -Name txtScriptBlock  -Column 1 -Row ($Row+=1) -MinHeight 141 -MinWidth 336 -AcceptsReturn:$true -HorizontalScrollBarVisibility Auto -VerticalScrollBarVisibility Auto)
       New-Label "Script _Block" -Row $Row -Target $target
       New-CheckBox "_Add to ISE Profile" -Name chkProfile -Row ($Row+=1)
       New-StackPanel -Orientation Horizontal  -Column 1 -Row ($Row+=1) -HorizontalAlignment Right {
          New-Button "_Save" -Name btnSave -Width 75 -Margin "0,0,5,0" -IsDefault -On_Click {
             if ($txtName.Text.Trim() -eq "" -or $txtShortcut.text.Trim() -eq "" -or $txtScriptBlock.text.Trim() -eq "") {
                 $Warning.Content = "You must supply all parameters"
             } else {
                $menuItems = $psise.CurrentPowerShellTab.AddOnsMenu.Submenus | Select -ExpandProperty DisplayName
                if ($menuItems -Contains $txtName.Text) {
                   $Warning.Content = "Select another Name for the menu"
                   return
                }            
                
                try {
                    $ScriptBlock = [ScriptBlock]::Create($txtScriptBlock.Text)
                    $psISE.CurrentPowerShellTab.AddOnsMenu.SubMenus.Add($txtName.Text,$ScriptBlock,$txtShortcut.Text) | Out-Null
                }
                catch {
                   $Warning.Content = "Fatal Error Creating MenuItem:`n$_"
                   return
                }
                if ($chkProfile.IsChecked) {
                   Add-Content -Path $profile -Value $profileText
                }
                $window.Close()
             }
          }
          New-Button "Cancel" -Name btnCancel -Width 75 -IsCancel
       }
    } -show -On_Load { $txtName.Focus() }
 }
 $psISE.CurrentPowerShellTab.AddOnsMenu.SubMenus.Add("New ISE menu",{New-ISEMenu},"ALT+M") | Out-Null
`

