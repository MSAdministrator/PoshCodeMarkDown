---
Author: jeff patton
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2953
Published Date: 2012-09-13t14
Archived Date: 2012-01-14t07
---

# psiselibrary - 

## Description

i’ve been doing some work lately with powershell add-ons and figured i’d add a couple that i’ve been working on.

## Comments



## Usage



## TODO



## function

`replace-tabswithspace`

## Code

`#
 #
 Function Replace-TabsWithSpace
 {
     <#
         .SYNOPSIS
             Replaces a tab character with 4 spaces
         .DESCRIPTION
             This function examines the selected text in the PSIE SelectedText property and every tab
             character that is found is replaced with 4 spaces.
         .PARAMETER SelectedText
             The current contents of the SelectedText property
         .PARAMETER InstallMenu
             Specifies if you want to install this as a PSIE add-on menu
         .EXAMPLE
             Replace-TabsWithSpace -InstallMenu $true
             
             Description
             -----------
             Installs the function as a menu item.
         .NOTES
             This was written specifically for me, I had some code originally created in Notepad++ that
             used actual tabs, later I changed that to spaces, but on occasion I come accross something
             that doesn't tab shift like it should. Since I've been doing some PowerShell ISE stuff lately
             I decided to write a little function that works as an Add-On menu.
         .LINK
     #>
     Param
     (
         $SelectedText = $psISE.CurrentFile.Editor.SelectedText,
         $InstallMenu
     )
     Begin
     {
         if ($InstallMenu)
         {
             Write-Verbose "Try to install the menu item, and error out if there's an issue."
             try
             {
                 $psISE.CurrentPowerShellTab.AddOnsMenu.SubMenus.Add("Replace Tabs with Space",{Replace-TabsWithSpace},"Ctrl+Alt+R") | Out-Null
                 }
             catch
             {
                 Return $Error[0].Exception
                 }
             }
         }
     Process
     {
         Write-Verbose "Try and find the tab character in the selected PSISE text, return an error if there's an issue."
         try
         {
             $psISE.CurrentFile.Editor.InsertText($SelectedText.Replace("`t","    "))
             }
         catch
         {
             Return $Error[0].Exception
             }
         }
     End
     {
         }
     }
 
 Function New-CommentBlock
 {
     <#
         .SYNOPSIS
             Inserts a full comment block
         .DESCRIPTION
             This function inserts a full comment block that is formatted the
             way I format all my comment blocks.
         .PARAMETER InstallMenu
             Specifies if you want to install this as a PSIE add-on menu
         .EXAMPLE
             New-CommentBlock -InstallMenu $true
             
             Description
             -----------
             Installs the function as a menu item.
         .NOTES
             FunctionName : New-CommentBlock
             Created by   : Jeff Patton
             Date Coded   : 09/13/2011 12:28:10
         .LINK
      #>
     Param
     (
         $InstallMenu
     )
     Begin
     {
         $CommentBlock = @(
             "    <#`n"
             "       .SYNOPSIS`n"
             "       .DESCRIPTION`n"
             "       .PARAMETER`n"
             "       .EXAMPLE`n"
             "       .NOTES`n"
             "           FunctionName : `n"
             "           Created by   : $($env:username)`n"
             "           Date Coded   : $(Get-Date)`n"
             "       .LINK`n"
             "    #>`n")
         if ($InstallMenu)
         {
             Write-Verbose "Try to install the menu item, and error out if there's an issue."
             try
             {
                 $psISE.CurrentPowerShellTab.AddOnsMenu.SubMenus.Add("Insert comment block",{New-CommentBlock},"Ctrl+Alt+C") | Out-Null
                 }
             catch
             {
                 Return $Error[0].Exception
                 }
             }
         }
     Process
     {
         if (!$InstallMenu)
         {
             Write-Verbose "Don't insert a comment if we're installing the menu"
             try
             {
                 Write-Verbose "Create a new comment block, return an error if there's an issue."
                 $psISE.CurrentFile.Editor.InsertText($CommentBlock)
                 }
             catch
             {
                 Return $Error[0].Exception
                 }
             }
         }
     End
     {
         }
     }
 
 Function New-Script
 {
     <#
         .SYNOPSIS
             Create a new blank script
         .DESCRIPTION
             This function creates a new blank script based on my original template.ps1
         .PARAMETER InstallMenu
             Specifies if you want to install this as a PSIE add-on menu
         .PARAMETER ScriptName
             This is the name of the new script.
         .EXAMPLE
             New-Script -ScriptName "New-ImprovedScript"
             
             Description
             -----------
             This example shows calling the function with the ScriptName parameter
         .EXAMPLE
             New-Script -InstallMenu $true
             
             Description
             -----------
             Installs the function as a menu item.
         .NOTES
             FunctionName : New-Script
             Created by   : Jeff Patton
             Date Coded   : 09/13/2011 13:37:24
         .LINK
      #>
     Param
     (
         $InstallMenu,
         $ScriptName
     )
     Begin
     {
         $TemplateScript = @(
         "<#`n"
         "   .SYNOPSIS`n"
         "       Template script`n"
         "   .DESCRIPTION`n"
         "       This script sets up the basic framework that I use for all my scripts.`n"
         "   .PARAMETER`n"
         "   .EXAMPLE`n"
         "   .NOTES`n"
         "       ScriptName : $($ScriptName)`n"
         "       Created By : $($env:Username)`n"
         "       Date Coded : $(Get-Date)`n"
         "       ScriptName is used to register events for this script`n"
         "       LogName is used to determine which classic log to write to`n"
         "`n"        
         "       ErrorCodes`n"
         "           100 = Success`n"
         "           101 = Error`n"
         "           102 = Warning`n"
         "           104 = Information`n"
         "   .LINK`n"
         "#>`n"
         "Param`n"
         "   (`n"
         "`n"    
         "   )`n"
         "Begin`n"
         "   {`n"
         "       `$ScriptName = `$MyInvocation.MyCommand.ToString()`n"
         "       `$LogName = `"Application`"`n"
         "       `$ScriptPath = `$MyInvocation.MyCommand.Path`n"
         "       `$Username = `$env:USERDOMAIN + `"\`" + `$env:USERNAME`n"
         "`n"
         "       New-EventLog -Source `$ScriptName -LogName `$LogName -ErrorAction SilentlyContinue`n"
         "`n"
         "       `$Message = `"Script: `" + `$ScriptPath + `"``nScript User: `" + `$Username + `"``nStarted: `" + (Get-Date).toString()`n"
         "       Write-EventLog -LogName `$LogName -Source `$ScriptName -EventID `"104`" -EntryType `"Information`" -Message `$Message`n"
         "`n"
         "       }`n"
         "Process`n"
         "   {`n"
         "       }`n"
         "End`n"
         "   {`n"
         "       `$Message = `"Script: `" + `$ScriptPath + `"``nScript User: `" + `$Username + `"``nFinished: `" + (Get-Date).toString()`n"
         "       Write-EventLog -LogName `$LogName -Source `$ScriptName -EventID `"104`" -EntryType `"Information`" -Message `$Message	`n"
         "       }`n")
         if ($InstallMenu)
         {
             Write-Verbose "Try to install the menu item, and error out if there's an issue."
             try
             {
                 $psISE.CurrentPowerShellTab.AddOnsMenu.SubMenus.Add("New blank script",{New-Script},"Ctrl+Alt+S") | Out-Null
                 }
             catch
             {
                 Return $Error[0].Exception
                 }
             }
 
         }
     Process
     {
         if (!$InstallMenu)
         {
             Write-Verbose "Don't create a script if we're installing the menu"
             try
             {
                 Write-Verbose "Create a new blank tab for the script"
                 $NewScript = $psISE.CurrentPowerShellTab.Files.Add()
                 Write-Verbose "Create a new empty script, return an error if there's an issue."
                 $NewScript.Editor.InsertText($TemplateScript)
                 $NewScript.Editor.InsertText(($NewScript.Editor.Select(22,1,22,2) -replace " ",""))
                 $NewScript.Editor.InsertText(($NewScript.Editor.Select(26,1,26,2) -replace " ",""))
                 $NewScript.Editor.InsertText(($NewScript.Editor.Select(40,1,40,2) -replace " ",""))
                 $NewScript.Editor.InsertText(($NewScript.Editor.Select(43,1,43,2) -replace " ",""))
                 $NewScript.Editor.Select(1,1,1,1)
                 $NewScript.SaveAs("$((Get-Location).Path)\$($ScriptName)")
                 }
             catch
             {
                 Return $Error[0].Exception
                 }
             }
         }
     End
     {
         Return $NewScript
         }
     }
 
 Function New-Function
 {
     <#
         .SYNOPSIS
             Create a new function
         .DESCRIPTION
             This function creates a new function that wraps the selected text inside
             the Process section of the body of the function.
         .PARAMETER SelectedText
             Currently selected code that will become a function
         .PARAMETER InstallMenu
             Specifies if you want to install this as a PSIE add-on menu
         .PARAMETER FunctionName
             This is the name of the new function.
         .EXAMPLE
             New-Function -FunctionName "New-ImprovedFunction"
             
             Description
             -----------
             This example shows calling the function with the FunctionName parameter
         .EXAMPLE
             New-Function -InstallMenu $true
             
             Description
             -----------
             Installs the function as a menu item.
         .NOTES
             FunctionName : New-Function
             Created by   : Jeff Patton
             Date Coded   : 09/13/2011 13:37:24
         .LINK
      #>
     Param
     (
         $SelectedText = $psISE.CurrentFile.Editor.SelectedText,
         $InstallMenu,
         $FunctionName
     )
     Begin
     {
         $TemplateFunction = @(
         "Function $FunctionName`n"
         "   <#`n"
         "       .SYNOPSIS`n"
         "       .DESCRIPTION`n"
         "       .PARAMETER`n"
         "       .EXAMPLE`n"
         "       .NOTES`n"
         "           FunctionName : $FunctionName`n"
         "           Created by   : $($env:username)`n"
         "           Date Coded   : $(Get-Date)`n"
         "       .LINK`n"
         "   #>`n"
         "Param`n"
         "    (`n"
         "    )`n"
         "Begin`n"
         "{`n"
         "    }`n"
         "Process`n"
         "{`n"
         "$($SelectedText)`n"
         "    }`n"
         "End`n"
         "{`n"
         "    }`n")
         if ($InstallMenu)
         {
             Write-Verbose "Try to install the menu item, and error out if there's an issue."
             try
             {
                 $psISE.CurrentPowerShellTab.AddOnsMenu.SubMenus.Add("New function",{New-Function},"Ctrl+Alt+S") | Out-Null
                 }
             catch
             {
                 Return $Error[0].Exception
                 }
             }
 
         }
     Process
     {
         if (!$InstallMenu)
         {
             Write-Verbose "Don't create a function if we're installing the menu"
             try
             {
                 Write-Verbose "Create a new empty function, return an error if there's an issue."
                 $psISE.CurrentFile.Editor.InsertText($TemplateFunction)
                 }
             catch
             {
                 Return $Error[0].Exception
                 }
             }
         }
     End
     {
         }
     }
`

