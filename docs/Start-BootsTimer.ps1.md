---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 1239
Published Date: 
Archived Date: 2009-08-02t06
---

# start-bootstimer - 

## Description

updated for powerboots 0.2

## Comments



## Usage



## TODO



## function

`start-bootstimer`

## Code

`#
 #
 Add-BootsFunction -Type "System.Windows.Threading.DispatcherTimer"
 
 function Start-BootsTimer {
 #.Syntax
 #.Description
 #.Parameter EndMessage
 #.Parameter StartMessage
 
 #.Parameter Minutes
 #.Parameter Seconds
 #.Parameter Hours
 
 #.Parameter SoundFile
 #.Parameter FontSize
 #.Parameter SingleAlarm
 
 #.Example
 #
 #
 #.Example
 #
 #
 #.Example
 #
 [CmdletBinding(DefaultParameterSetName="Times")]
 PARAM( 
    [Parameter(Position=2,ParameterSetName="Times",Mandatory=$false)]
    [Parameter(Position=1,ParameterSetName="Reasons", Mandatory=$true)]
    [string]$EndMessage
 ,  
    [Parameter(Position=2,ParameterSetName="Reasons", Mandatory=$false)]
    [string]$StartMessage
 ,  
    [Parameter(Position=3,ParameterSetName="Reasons", Mandatory=$false)]
    [int]$minutes   = 0
 ,
    [Parameter(Position=4,ParameterSetName="Reasons", Mandatory=$false)]
    [Parameter(Position=1,ParameterSetName="Times", Mandatory=$true)]
    [int]$seconds   = 0
 ,
    [Parameter()]
    [int]$hours     = 0
 ,
    [Parameter()]
    $SoundFile = "$env:SystemRoot\Media\notify.wav"
 ,
    [Parameter()]
    $FontSize = 125
 , 
    [Parameter()]
    [Switch]$SingleAlarm
 )
 
 if(($seconds + $Minutes + $hours) -eq 0) { $seconds = 10 }
 
 $start = [DateTime]::Now
 
 $TimerStuff = @{}
 $TimerStuff["seconds"] = [Math]::Abs($seconds) + [Math]::Abs([int]($minutes*60)) + [Math]::Abs([int]($hours*60*60))
 $TimerStuff["TimerEnd"] = $start.AddSeconds( $TimerStuff["seconds"] )
 $TimerStuff["SingleAlarm"] = $SingleAlarm
 
 if(Test-Path $soundFile) {
    $TimerStuff["Sound"] = new-Object System.Media.SoundPlayer
    $TimerStuff["Sound"].SoundLocation=$SoundFile
 }
 if($EndMessage -or $StartMessage) {
    $TimerStuff["Voice"] = new-object -com SAPI.SpVoice
 }
 
 if($EndMessage) {
    $TimerStuff["EndMessage"] = $EndMessage
 }
 if($StartMessage) {
    $null = $TimerStuff["Voice"].Speak( $StartMessage, 1 )
 }
 
 $TimerStuff["FontSize"] = $FontSize
 
 PowerBoots\New-BootsWindow -WindowStyle None -AllowsTransparency -Tag $TimerStuff {
    Param($window)
    TextBlock "00:00:00" -FontSize $window.Tag.FontSize -FontFamily Impact -margin 20   `
             -BitmapEffect $(OuterGlowBitmapEffect -GlowColor White -GlowSize 15)    `
             -Foreground $(LinearGradientBrush -Start "1,1" -End "0,1" {
                            GradientStop -Color Black -Offset 0.0
                            GradientStop -Color Black -Offset 0.95
                            GradientStop -Color Red -Offset 1.0
                            GradientStop -Color Red -Offset 1.0
 
    $window.Tag["Timer"] = DispatcherTimer -Tag $window -Interval "0:0:0.05" -On_Tick { 
       trap { 
          write-host "Error: $_" -fore Red 
          write-host $($_.InvocationInfo.PositionMessage) -fore Red 
          write-host $($_.Exception.StackTrace | Out-String) -fore Red 
       }
 
       $remain = 100
       if($this.Tag.Tag.TimerEnd -and $this.Tag.Content.Foreground.GradientStops.Count -ge 3) {
          Write-Verbose $($this.Tag.Tag|Out-String) #-fore magenta
          $diff = $this.Tag.Tag.TimerEnd - [DateTime]::Now;
          if($diff.Ticks -ge 0) {
             $this.Tag.Content.Text = ([DateTime]$diff.Ticks).ToString(" HH:mm.ss")
          } else {
             $this.Tag.Content.Text = ([DateTime][Math]::Abs($diff.Ticks)).ToString("-HH:mm.ss")
          }
          
          $remain = $diff.TotalSeconds / $this.tag.tag.seconds
          Write-Verbose "Remain: $remain or $($diff.TotalSeconds) of $($this.tag.tag.seconds)"
       } else { Write-Host "Wahat!" }
       if($remain -le 0) {
          if($this.Interval.Seconds -eq 0) {
             $this.Interval = [TimeSpan]"0:0:2"
             $this.tag.Add_MouseDown( { $this.tag.Close() } ) 
             if($this.tag.tag["SingleAlarm"]) { $this.tag.Close() }
          }
          & {
             if($this.tag.Tag["Sound"]) {
                $this.tag.Tag["Sound"].Play()
             } else {
                [System.Media.SystemSounds]::Exclamation.Play()
             }
             if($this.tag.Tag["EndMessage"]) {
                $null = $this.tag.Tag["Voice"].Speak( $this.tag.Tag["EndMessage"], 1 )
             }
          }
       }
    }
    $window.Add_Closed( { Write-Host "Shutting Down $this"; $this.Tag.Timer.Stop() } )
    
 } -On_MouseDown { 
    if($_.ChangedButton -eq "Left") { $this.DragMove()  }
 } -Async -Topmost -Background Transparent -ShowInTaskbar:$False -Passthru | 
 ForEach { Invoke-BootsWindow $_ { $BootsWindow.Tag.Timer.Start() } }
 
 }
`

