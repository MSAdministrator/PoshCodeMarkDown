---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5295
Published Date: 2015-07-09t23
Archived Date: 2015-04-24t02
---

# show-consolemenu - 

## Description

shows a vertical “menu” in the console and allows you to pick numeric items from it.

## Comments



## Usage



## TODO



## function

`show-consolemenu`

## Code

`#
 #
 function Show-ConsoleMenu {
 #.Synopsis
 #.Description
 #.Example
 #
 #.Example
 #
 param(
    [Parameter(ValueFromPipeline=$true,Position=2)]
    [Alias("InputObject")]
    [PSObject[]]$Choices,
    [Parameter(Position=1)]
    [Alias("Title")]
    [string]$Caption,
    [Parameter(Position=3)]
    [ScriptBlock]$Expression=({$_}),
    [Alias("Footer")]
    [string]$Prompt,
    [int]$indentLeft=8,
    [Switch]$Passthru,
    [Switch]$UseBufferBox,
    [Switch]$MultiSelect
 )
 begin {
    $allchoices = New-Object System.Collections.Generic.List[PSObject]
 }
 process {
    if($choices) {
       $allchoices.AddRange($choices)
    }
 }
 end {
    $Format = "{0:D1}"; 
    $Digits = ($allchoices.Count + 1).ToString().Length
    $Format = "{0:D${Digits}}" 
    
    for($i=0; $i -lt $allchoices.Count; $i++) {
       $allchoices[$i] = Add-Member -Input $allchoices[$i] -Type NoteProperty -Name ConsoleMenuKey -Value $($format -f ($i+1)) -Passthru
    }
 
    Write-Debug "There are $($allChoices.Count) choices, so we'll use $Digits digits"
    $menu = $allchoices | Format-Table -HideTableHeader @{E="ConsoleMenuKey";A="Right";W=$indentLeft}, @{E=$Expression;A="Left"} -Force | Out-String
    $menu = $menu.TrimEnd() + "`n" 
 
    if($UseBufferBox) {
       Out-Buffer ("`n" + (" " * ($indentLeft/2)) + $Caption + "`n")  -ForegroundColor $Host.PrivateData.VerboseForegroundColor  -BackgroundColor $Host.PrivateData.VerboseBackgroundColor
       Out-Buffer $menu
    } else {
       Write-Host ("`n" + (" " * ($indentLeft/2)) + $Caption + "`n")  -ForegroundColor $Host.PrivateData.VerboseForegroundColor  -BackgroundColor $Host.PrivateData.VerboseBackgroundColor
       Write-Host $menu
    }
    
    do {
       if($Prompt) {
          if($UseBufferBox) {
             Out-Buffer $Prompt -ForegroundColor $Host.PrivateData.VerboseForegroundColor  -BackgroundColor $Host.PrivateData.VerboseBackgroundColor
          } else {
             Write-Host $Prompt -ForegroundColor $Host.PrivateData.VerboseForegroundColor  -BackgroundColor $Host.PrivateData.VerboseBackgroundColor
          }
       }      
       [string]$PreviousKeys=""
       do { 
          $Key = $Host.UI.RawUI.ReadKey("IncludeKeyDown,NoEcho").Character
          try { 
             [int][string]$choice = "${PreviousKeys}${Key}"
             $index = $choice - 1
             $PreviousKeys = "${PreviousKeys}${Key}"
             Write-Host $Key -NoNewline
          } catch { 
             if(63 -eq [int][char]$Key) {
                if($UseBufferBox) {
                   Out-Buffer $menu
                } else {
                   Write-Host $menu
                }
             } elseif(13,27,0 -notcontains [int][char]$Key) {
                $warning = "You must type only numeric characters (hit Esc to exit)"
                if($UseBufferBox) {
                   Out-Buffer $warning -ForegroundColor $Host.PrivateData.WarningForegroundColor -BackgroundColor $Host.PrivateData.WarningBackgroundColor
                } else {
                   Write-Warning $warning
                }
             }
          }
       } while( $PreviousKeys.Length -lt $Digits -and (13,27 -notcontains [int][char]$Key))
 
       if($PreviousKeys.Length -and $allchoices.Count -gt $index -and $index -ge 0) {
          Write-Host
          if($Passthru) { 
             $allchoices[$index] 
          } else { 
             $choice
          }
       }
    } while($key -ne [char]13 -and $MultiSelect)
 }}
`

