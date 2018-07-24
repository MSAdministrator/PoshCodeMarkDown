---
Author: oisin grehan ( http
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5998
Published Date: 2016-09-01t19
Archived Date: 2016-08-03t16
---

# powershell cmatrix - 

## Description

a pure console screen saver in the vein of the popular cmatrix x-term screensaver. powershell 2.0 module, see blog post

## Comments



## Usage



## TODO



## module

`new-size`

## Code

`#
 #
 Set-StrictMode -off
 
 #
 #
 #
 #
 
 if ($host.ui.rawui.windowsize -eq $null) {
     write-warning "Sorry, I only work in a true console host like powershell.exe."
     throw
 }
 
 #
 #
 
 function New-Size {
     param([int]$width, [int]$height)
     
     new-object System.Management.Automation.Host.Size $width,$height
 }
 
 function New-Rectangle {
     param(
         [int]$left,
         [int]$top,
         [int]$right,
         [int]$bottom
     )
     
     $rect = new-object System.Management.Automation.Host.Rectangle
     $rect.left= $left
     $rect.top = $top
     $rect.right =$right
     $rect.bottom = $bottom
     
     $rect
 }
 
 function New-Coordinate {
     param([int]$x, [int]$y)
     
     new-object System.Management.Automation.Host.Coordinates $x, $y
 }
 
 function Get-BufferCell {
     param([int]$x, [int]$y)
     
     $rect = new-rectangle $x $y $x $y
     
     [System.Management.Automation.Host.buffercell[,]]$cells = $host.ui.RawUI.GetBufferContents($rect)    
     
     $cells[0,0]
 }
 
 function Set-BufferCell {
     [outputtype([System.Management.Automation.Host.buffercell])]
     param(
         [int]$x,
         [int]$y,
         [System.Management.Automation.Host.buffercell]$cell
     )
     
     $rect = new-rectangle $x $y $x $y
         
     get-buffercell $x $y
 
     $host.ui.rawui.SetBufferContents($rect, $cell)
 }
 
 function New-BufferCell {
     param(
         [string]$Character,
         [consolecolor]$ForeGroundColor = $(get-buffercell 0 0).foregroundcolor,
         [consolecolor]$BackGroundColor = $(get-buffercell 0 0).backgroundcolor,
         [System.Management.Automation.Host.BufferCellType]$BufferCellType = "Complete"
     )
     
     $cell = new-object System.Management.Automation.Host.BufferCell
     $cell.Character = $Character
     $cell.ForegroundColor = $foregroundcolor
     $cell.BackgroundColor = $backgroundcolor
     $cell.BufferCellType = $buffercelltype
     
     $cell
 }
 
 function log {
     param($message)
     [diagnostics.debug]::WriteLine($message, "PS ScreenSaver")
 }
 
 #
 #
 
 function Start-CMatrix {
     param(
         [int]$maxcolumns = 8,
         [int]$frameWait = 100
     )
 
     $script:winsize = $host.ui.rawui.WindowSize
     $script:framenum = 0
         
     $prevbg = $host.ui.rawui.BackgroundColor
     $host.ui.rawui.BackgroundColor = "black"
     cls
     
     $done = $false        
     
     while (-not $done) {
 
         Write-FrameBuffer -maxcolumns $maxcolumns
 
         Show-FrameBuffer
         
         sleep -milli $frameWait
         
         $done = $host.ui.rawui.KeyAvailable        
     }
     
     $host.ui.rawui.BackgroundColor = $prevbg
     cls
 }
 
 function Write-FrameBuffer {
     param($maxColumns)
 
     if ($columns.count -lt $maxcolumns) {
         
         if ((get-random -min 0 -max 10) -lt 5) {
             
             do { 
                 $x = get-random -min 0 -max ($winsize.width - 1)
             } while ($columns.containskey($x))
             
             $columns.add($x, (new-column $x))
             
         }
     }
     
     $script:framenum++
 }
 
 function Show-FrameBuffer {
     param($frame)
     
     $completed=@()
     
     foreach ($entry in $columns.getenumerator()) {
         
         $column = $entry.value
     
         if (-not $column.step()) {            
             $completed += $entry.key
         }
     }
     
     foreach ($key in $completed) {
         $columns.remove($key)
     }    
 }
 
 function New-Column {
     param($x)
     
     
     new-module -ascustomobject -name "col_$x" -script {
         param(
             [int]$startx,
             [PSModuleInfo]$parentModule
          )
         
         $script:xpos = $startx
         $script:ylimit = $host.ui.rawui.WindowSize.Height
 
         [int]$script:head = 1
         [int]$script:fade = 0
         [int]$script:fadelen = [math]::Abs($ylimit / 3)
         
         $script:fadelen += (get-random -min 0 -max $fadelen)
         
         function Step {
             
             if ($head -lt $ylimit) {
 
                 & $parentModule Set-BufferCell $xpos $head (
                     & $parentModule New-BufferCell -Character `
                         ([char](get-random -min 65 -max 122)) -Fore white) > $null
                 
                 & $parentModule Set-BufferCell $xpos ($head - 1) (
                     & $parentModule New-BufferCell -Character `
                         ([char](get-random -min 65 -max 122)) -Fore green) > $null
                 
                 $script:head++
             }
             
             if ($head -gt $fadelen) {
 
                 & $parentModule Set-BufferCell $xpos $fade (
                     & $parentModule New-BufferCell -Character `
                         ([char](get-random -min 65 -max 122)) -Fore darkgreen) > $null
                     
                 $script:fade++
             }
             
             if ($fade -lt $ylimit) {
                 return $true
             }
                         
             $false            
         }
                 
         Export-ModuleMember -function Step
         
     } -args $x, $executioncontext.sessionstate.module
 }
 
 function Start-ScreenSaver {
     
     
     Start-CMatrix -max 8 -frame 50
 }
 
 function Register-Timer {
 
     if ($_ssdisabled) {
         return
     }
     
     if (-not (Test-Path variable:global:_ssjob)) {
         
         $global:_ssjob = Register-ObjectEvent $_sstimer elapsed -action {
             
             $global:_sscount++;
             $global:_sssrcid = $event.sourceidentifier
                 
             if ($_sscount -eq $_sstimeout) {
                 
                 Unregister-Event -sourceidentifier $_sssrcid
                 Remove-Variable _ssjob -scope Global
                            
                 sleep -seconds 1
                      
                 Start-ScreenSaver
             }
 
         }
     }
 }
 
 function Enable-ScreenSaver {
     
     if (-not $_ssdisabled) {
         write-warning "Screensaver is not disabled."
         return
     }
     
     $global:_ssdisabled = $false    
 }
 
 function Disable-ScreenSaver {
 
     if ((Test-Path variable:global:_ssjob)) {
 
         $global:_ssdisabled = $true
         Unregister-Event -SourceIdentifier $_sssrcid        
         Remove-Variable _ssjob -Scope global        
 
     } else {
         write-warning "Screen saver is not enabled."
     }
 }
 
 function Get-ScreenSaverTimeout {
     new-timespan -seconds $global:_sstimeout
 }
 
 function Set-ScreenSaverTimeout {
     [cmdletbinding(defaultparametersetname="int")]
     param(
         [parameter(position=0, mandatory=$true, parametersetname="int")]
         [int]$Seconds,
         
         [parameter(position=0, mandatory=$true, parametersetname="timespan")]
         [Timespan]$Timespan
     )
     
     if ($pscmdlet.parametersetname -eq "int") {
         $timespan = new-timespan -seconds $Seconds
     }
     
     if ($timespan.totalseconds -lt 1) {
         throw "Timeout must be greater than 0 seconds."
     }
     
     $global:_sstimeout = $timespan.totalseconds
 }
 
 #
 #
 
 
 [int]$global:_sscount = 0
 
 
 $self = $ExecutionContext.SessionState.Module
 $function:global:prompt = $self.NewBoundScriptBlock(
     [scriptblock]::create(
         ("{0}`n`$global:_sscount = 0`nRegister-Timer" `
             -f ($global:_ssprompt = gc function:prompt))))
 
 $global:_sstimer = new-object system.timers.timer
 $_sstimer.AutoReset = $true
 $_sstimer.start()
 
 $global:_ssdisabled = $true
 
 $ExecutionContext.SessionState.Module.OnRemove = { 
     
     $function:global:prompt = [scriptblock]::Create($_ssprompt)
     
     if ($_sssrcid) {
         Unregister-Event -SourceIdentifier $_sssrcid
     }
     
     $_sstimer.Dispose()
     
     remove-variable _ss* -scope global
 }
 
 Export-ModuleMember -function Start-ScreenSaver, Get-ScreenSaverTimeout, `
     Set-ScreenSaverTimeout, Enable-ScreenSaver, Disable-ScreenSaver
`

