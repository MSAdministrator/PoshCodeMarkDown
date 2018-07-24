---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6062
Published Date: 2016-10-21t22
Archived Date: 2016-08-26t03
---

# my profile.ps1 - 

## Description

this is my latest profile script … it contains many things specific to my setup, but they should all be available here on poshcode.org

## Comments



## Usage



## TODO



## script

`reset-module`

## Code

`#
 #
 trap { Write-Warning ($_.ScriptStackTrace | Out-String) }
 
 $Script:TraceVerboseTimer = New-Object System.Diagnostics.Stopwatch
 $Script:TraceVerboseTimer.Start()
 
 try { $NOCONSOLE = $FALSE; [System.Console]::Clear() } catch { $NOCONSOLE = $TRUE }
 
 Set-ExecutionPolicy AllSigned Process
 
 Import-Module $PSScriptRoot\Modules\Environment, Microsoft.PowerShell.Management, Microsoft.PowerShell.Security, Microsoft.PowerShell.Utility
 
 Add-Type -Assembly PresentationCore, WindowsBase
 try {
     $global:SHIFTED = [System.Windows.Input.Keyboard]::IsKeyDown([System.Windows.Input.Key]::LeftShift) -OR
                   [System.Windows.Input.Keyboard]::IsKeyDown([System.Windows.Input.Key]::RightShift)
 } catch {
     $global:SHIFTED = $false
 }
 if($SHIFTED) { $VerbosePreference = "Continue" }
 
 if($Host.Name -eq "ConsoleHost") {
     $Host.PrivateData.ErrorForegroundColor    = "DarkRed"
     $Host.PrivateData.WarningForegroundColor  = "DarkYellow"
     $Host.PrivateData.DebugForegroundColor    = "Green"
     $Host.PrivateData.VerboseForegroundColor  = "Cyan"
     $Host.PrivateData.ProgressForegroundColor = "Yellow"
     $Host.PrivateData.ProgressBackgroundColor = "DarkMagenta"
     if($PSProcessElevated) {
         $Host.UI.RawUI.BackgroundColor = "DarkGray"
     }
 } elseif($Host.Name -eq "Windows PowerShell ISE Host") {
     $Host.PrivateData.ErrorForegroundColor    = "DarkRed"
     $Host.PrivateData.WarningForegroundColor  = "Gold"
     $Host.PrivateData.DebugForegroundColor    = "Green"
     $Host.PrivateData.VerboseForegroundColor  = "Cyan"
     if($PSProcessElevated) {
         $Host.UI.RawUI.BackgroundColor = "DarkGray"
     }
 }
 
 Trace-Message "Microsoft.PowerShell.* Modules Imported" -Stopwatch $TraceVerboseTimer
 
 Set-Variable ProfileDir (Split-Path $MyInvocation.MyCommand.Path -Parent) -Scope Global -Option AllScope, Constant -ErrorAction SilentlyContinue
 Set-Variable LiveID (
         [System.Security.Principal.WindowsIdentity]::GetCurrent().Groups |
         Where-Object { $_.Value -match "^S-1-11-96" }
     ).Translate([System.Security.Principal.NTAccount]).Value  -Scope Global -Option AllScope, Constant -ErrorAction SilentlyContinue
 Set-Variable ReReverse @('(?sx) . (?<=(?:.(?=.*$(?<=((.) \1?))))*)', '$2') -Scope Global -Option AllScope, Constant -ErrorAction SilentlyContinue
 
 ###################################################################################################
 [string[]]$folders = Get-ChildItem $ProfileDir\Tool[s], $ProfileDir\Utilitie[s], $ProfileDir\Scripts\*,$ProfileDir\Script[s] -ad | % FullName
 
 $folders += [System.Runtime.InteropServices.RuntimeEnvironment]::GetRuntimeDirectory()
 $folders += Set-AliasToFirst -Alias "msbuild" -Path 'C:\Program Files (x86)\MSBuild\*\Bin\MsBuild.exe' -Description "Visual Studio's MsBuild" -Force -Passthru
 $folders += Set-AliasToFirst -Alias "merge" -Path "C:\Program*Files*\Perforce\p4merge.exe","C:\Program*Files*\DevTools\Perforce\p4merge.exe" -Description "Perforce" -Force -Passthru
 $folders += Set-AliasToFirst -Alias "tf" -Path "C:\Program*Files*\*Visual?Studio*\Common7\IDE\TF.exe", "C:\Program*Files*\DevTools\*Visual?Studio*\Common7\IDE\TF.exe" -Description "Visual Studio" -Force -Passthru
 $folders += Set-AliasToFirst -Alias "Python","Python2","py2" -Path "C:\Python2*\python.exe", "D:\Python2*\python.exe" -Description "Python 2.x" -Force -Passthru
 $folders += Set-AliasToFirst -Alias "Python3","py3" -Path "C:\Python3*\python.exe", "D:\Python3*\python.exe" -Description "Python 3.x" -Force -Passthru
 Set-AliasToFirst -Alias "iis","iisexpress" -Path 'C:\Progra*\IIS*\IISExpress.exe' -Description "Personal Profile Alias"
 
 Trace-Message "Development aliases set"
 
 if(!(Test-Path Env:Editor)) {
    if($Editor = Get-Item 'C:\Progra*\Sublime*\sublime_text.exe','C:\Progra*\*\Sublime*\sublime_text.exe' | Sort VersionInfo.ProductVersion -Desc | Select-Object -First 1) {
       $folders += Split-Path $Editor
 
       function edit { start $Editor @( $Args + @("-n","-w")) }
       [Environment]::SetEnvironmentVariable("Editor", "'${Env:Editor}' -n -w", "User")
       Trace-Message "Env:Editor set: ${Env:Editor} "
    } else {
       Trace-Message -AsWarning "Sublime Text (edit command) is not available"
    }
 }
 
 $ENV:PATH = Select-UniquePath $folders ${Env:Path}
 Trace-Message "PATH Updated"
 
 $Env:PSModulePath = Select-UniquePath "$ProfileDir\Modules",(Get-SpecialFolder *Modules -Value),${Env:PSModulePath},"${Home}\Projects\Modules"
 Trace-Message "PSModulePath Updated "
 
 Update-TypeData   -PrependPath "$ProfileDir\Formats\Types.ps1xml"
 Trace-Message "Type Data Updated"
 
 Update-FormatData -PrependPath "$ProfileDir\Formats\Formats.ps1xml"
 Trace-Message "Format Data Updated"
 
 function Reset-Module ($ModuleName) { rmo $ModuleName; ipmo $ModuleName -force -pass | ft Name, Version, Path -Auto }
 
 function qq {param([Parameter(ValueFromRemainingArguments=$true)][string[]]$q)$q}
 
 Set-Alias   say          Speech\Out-Speech         -Option Constant, ReadOnly, AllScope -Description "Personal Profile Alias"
 Set-Alias   gph          Get-PerformanceHistory    -Option Constant, ReadOnly, AllScope -Description "Personal Profile Alias"
 
 if($ProfileDir -ne (Get-Location)) {
    Push-Location $ProfileDir
 }
 
 New-PSDrive Documents FileSystem (Get-SpecialFolder MyDocuments -Value)
 
 if( ($OneDrive = (Get-ItemProperty HKCU:\Software\Microsoft\OneDrive UserFolder -EA 0).UserFolder) -OR
     ($OneDrive = (Get-ItemProperty HKCU:\Software\Microsoft\SkyDrive UserFolder -EA 0).UserFolder) -OR
     ($OneDrive = (Get-ItemProperty HKCU:\Software\Microsoft\Windows\CurrentVersion\SkyDrive UserFolder -EA 0).UserFolder)
   ) {
     New-PSDrive OneDrive FileSystem $OneDrive
     Trace-Message "OneDrive:\ mapped to $OneDrive"
 }
 if(Test-Path $Home\Projects) {
     New-PSDrive Projects FileSystem $Home\Projects
     New-PSDrive Modules FileSystem $Home\Projects\Modules
 }
 
 
 if($Host.Name -ne "Package Manager Host") {
   . Set-Prompt -Clean
   Trace-Message "Prompt fixed"
 }
 
 if($Host.Name -eq "ConsoleHost" -and !$NOCONSOLE) {
 
     if(-not (Get-Module PSReadLine)) { Import-Module PSReadLine }
 
     function Set-PSReadLineMyWay {
         param(
             $BackgroundColor = $(if($PSProcessElevated) { "DarkGray" } else { "Black" } )
         )
         $Host.UI.RawUI.BackgroundColor = $BackgroundColor
         $Host.UI.RawUI.ForegroundColor = "Gray"
 
         Set-PSReadlineOption -TokenKind Keyword -ForegroundColor Yellow -BackgroundColor $BackgroundColor
         Set-PSReadlineOption -TokenKind String -ForegroundColor Green -BackgroundColor $BackgroundColor
         Set-PSReadlineOption -TokenKind Operator -ForegroundColor DarkGreen -BackgroundColor $BackgroundColor
         Set-PSReadlineOption -TokenKind Variable -ForegroundColor DarkMagenta -BackgroundColor $BackgroundColor
         Set-PSReadlineOption -TokenKind Command -ForegroundColor DarkYellow -BackgroundColor $BackgroundColor
         Set-PSReadlineOption -TokenKind Parameter -ForegroundColor DarkCyan -BackgroundColor $BackgroundColor
         Set-PSReadlineOption -TokenKind Type -ForegroundColor Blue -BackgroundColor $BackgroundColor
         Set-PSReadlineOption -TokenKind Number -ForegroundColor Red -BackgroundColor $BackgroundColor
         Set-PSReadlineOption -TokenKind Member -ForegroundColor DarkRed -BackgroundColor $BackgroundColor
         Set-PSReadlineOption -TokenKind None -ForegroundColor White -BackgroundColor $BackgroundColor
         Set-PSReadlineOption -TokenKind Comment -ForegroundColor Black -BackgroundColor DarkGray
 
         Set-PSReadlineOption -EmphasisForegroundColor White -EmphasisBackgroundColor $BackgroundColor `
                              -ContinuationPromptForegroundColor DarkBlue -ContinuationPromptBackgroundColor $BackgroundColor `
                              -ContinuationPrompt (([char]183) + "  ")
     }
 
     Set-PSReadLineMyWay
     Set-PSReadlineKeyHandler -Key "Ctrl+Shift+R" -Functio ForwardSearchHistory
     Set-PSReadlineKeyHandler -Key "Ctrl+R" -Functio ReverseSearchHistory
 
     Set-PSReadlineKeyHandler Ctrl+M SetMark
     Set-PSReadlineKeyHandler Ctrl+Shift+M ExchangePointAndMark
 
     Set-PSReadlineKeyHandler Ctrl+K KillLine
     Set-PSReadlineKeyHandler Ctrl+I Yank
     Trace-Message "PSReadLine fixed"
 } else {
     Remove-Module PSReadLine -ErrorAction SilentlyContinue
     Trace-Message "PSReadLine skipped!"
 }
 
 ###################################################################################################
 $Script:QuoteDir = Join-Path (Split-Path $ProfileDir -parent) "@stuff\Quotes"
 if(Test-Path $Script:QuoteDir) {
     Set-Variable QuoteDir (Resolve-Path $QuoteDir) -Scope Global -Option AllScope -Description "Personal PATH Variable"
 
     function Get-Quote {
         param(
             $Path = "${QuoteDir}\attributed quotes.txt",
             [int]$Count=1
         )
         if(!(Test-Path $Path) ) {
             $Path = Join-Path ${QuoteDir} $Path
             if(!(Test-Path $Path) ) {
                 $Path = $Path + ".txt"
             }
         }
         Get-Content $Path | Where-Object { $_ } | Get-Random -Count $Count
     }
     
     if( Test-Path "${QuoteDir}\attributed quotes.txt" ) {
         Get-Quote | Write-Host -Foreground Yellow
     }
     
     Set-Alias gq Get-Quote
     Trace-Message "Random Quotes Loaded"
 }
 
 $ExecutionContext.SessionState.InvokeCommand.CommandNotFoundAction = {
     param( $CommandName, $CommandLookupEventArgs )
     if($CommandName.Contains([char]8211)) {
         $CommandLookupEventArgs.Command = Get-Command ( $CommandName -replace ([char]8211), ([char]45) ) -ErrorAction Ignore
     }
 }
 
 Trace-Message "Profile Finished!" -KillTimer
 
 Set-ExecutionPolicy RemoteSigned Process
 
`

