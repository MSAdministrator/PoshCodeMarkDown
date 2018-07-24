---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 6485
Published Date: 2016-08-21t17
Archived Date: 2016-10-24t03
---

# set-prompt.ps1 - 

## Description

my prompt function for ps 6.0.0.9 — there’s a bug causing an echo if you use write-host in your prompt on unix with ps 6 currently, so i rewrote my prompt to just output a string with escape sequences…

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 $e = ([char]27) + "["
 $global:ANSI = @{
    ESC = ([char]27) + "["
    Clear = "${e}0m"
    fg = @{
       Clear       = "${e}39m"
 
       Black       = "${e}30m";  DarkGray    = "${e}90m"
       DarkRed     = "${e}31m";  Red         = "${e}91m"
       DarkGreen   = "${e}32m";  Green       = "${e}92m"
       DarkYellow  = "${e}33m";  Yellow      = "${e}93m"
       DarkBlue    = "${e}34m";  Blue        = "${e}94m"
       DarkMagenta = "${e}35m";  Magenta     = "${e}95m"
       DarkCyan    = "${e}36m";  Cyan        = "${e}96m"
       Gray        = "${e}37m";  White       = "${e}97m"
    }
    bg = @{
       Clear       = "${e}49m"
       Black       = "${e}40m"; DarkGray    = "${e}100m"
       DarkRed     = "${e}41m"; Red         = "${e}101m"
       DarkGreen   = "${e}42m"; Green       = "${e}102m"
       DarkYellow  = "${e}43m"; Yellow      = "${e}103m"
       DarkBlue    = "${e}44m"; Blue        = "${e}104m"
       DarkMagenta = "${e}45m"; Magenta     = "${e}105m"
       DarkCyan    = "${e}46m"; Cyan        = "${e}106m"
       Gray        = "${e}47m"; White       = "${e}107m"
    }
 }
 
 $global:ANSI.fg.Default = $global:ANSI.fg."$($Host.UI.RawUI.ForegroundColor)"
 $global:ANSI.fg.Background = $global:ANSI.fg."$($Host.UI.RawUI.BackgroundColor)"
 $global:ANSI.bg.Default = $global:ANSI.bg."$($Host.UI.RawUI.BackgroundColor)"
 
 
 function global:prompt {
    $err = !$?
 
 
 
    try {
       $Host.UI.RawUI.WindowTitle = "{0} - {1} ({2})" -f $global:WindowTitlePrefix, (Convert-Path $pwd),  $pwd.Provider.Name
       [Environment]::CurrentDirectory = (Get-Location -PSProvider FileSystem).ProviderPath
    } catch {}
 
    $Nesting = "$GEAR" * $NestedPromptLevel
 
    $Stack = (Get-Location -Stack).count
 
    $(&{
       if($env:ConEmuANSI -eq "ON" -or $env:TERM -match "xterm") {
          $w = [Console]::BufferWidth
          $local:LastCommand = Get-History -Count 1
          $Elapsed = if($global:LastCommand.ID -ne $LastCommand.Id) {
             $global:LastCommand = $local:LastCommand
             $Duration = $LastCommand.EndExecutionTime - $LastCommand.StartExecutionTime
             if($Duration.TotalSeconds -ge 1.0) {
                "{0:h\:mm\:ss\.ffff}" -f $Duration
             } else {
                "{0}ms" -f $Duration.TotalMilliseconds
             }
          } else { '' }
          $ElapsedLength = [Math]::Max($Elapsed.Length,12)
          $ElapsedPadding = " " * ($ElapsedLength - $Elapsed.Length)
          $TimeStamp = "{0:h:mm:ss tt}" -f [DateTime]::Now
 
          if($Elapsed) {
             "${e}1A${e}${w}G${e}${ElapsedLength}D"
          }
          "${e}${w}G${e}$($TimeStamp.Length)D"
       }
 
          if($err) {
             $fg_ = $ANSI.fg.DarkRed
             $bg_ = $ANSI.bg.DarkRed
          } else {
             $fg_ = $ANSI.fg.DarkBlue
             $bg_ = $ANSI.bg.DarkBlue
          }
 
          "$($ANSI.fg.DarkYellow)$($ANSI.bg.DarkYellow)<$($ANSI.fg.White)$($ANSI.bg.DarkYellow)#$($myinvocation.historyID)${Nesting}"
          if($Stack) {
             "$($ANSI.fg.DarkYellow)$($ANSI.bg.DarkGray)$RIGHT"
          } else {
             "$($ANSI.fg.DarkYellow)$bg_$RIGHT"
          }
          
          "$($ANSI.fg.White)$bg_$($pwd.Drive.Name):${GT}$(Split-Path $pwd.Path -Leaf)"
          
          if($global:VcsStatusEnabled) {
             Write-VcsStatus
          } else {
             "$fg_$($ANSI.bg.Default)$RIGHT$($ANSI.fg.Background)$($ANSI.bg.Default)#>$($ANSI.Clear)"
          }
       }) -join ""
 }
`

