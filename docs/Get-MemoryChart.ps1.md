---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2309
Published Date: 
Archived Date: 2010-10-21t23
---

# get-memorychart - 

## Description

updated for current powerboot (latest chackin)

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #.Synopsis
 #.Description
 #.Parameter hosts
 #.Example
 #
 #.Example
 #
 #
 Param([string[]]$hosts = "localhost")
 
 Import-Module PowerBoots
 if(!(Get-Command JournalEntry -EA SilentlyContinue)) {
    Add-BootsFunction -Type System.Windows.Navigation.JournalEntry
 }
 if(!(Get-Command Chart -EA SilentlyContinue)) {
    Add-BootsContentProperty 'DataPoints', 'Series'
    Add-BootsFunction -Assembly "~\Documents\WindowsPowershell\Libraries\WPFVisifire.Charts.dll" 2>$Null
    Add-BootsFunction -Assembly "~\Documents\WindowsPowershell\Libraries\Transitionals.dll"
 }
 
 $global:comical = Get-Command Get-Comic -EA SilentlyContinue
 if($comical) {
    $global:comic = Get-Comic xkcd
    if(!$comic) { $comical = $false } else {
       $image = [system.drawing.image]::fromfile( $comic.FullName )
       $global:comicwidth = $image.Width
       $image.Dispose()
    }
 }
 
 #$window = Boots { Image -Source $xkcd -MinHeight 100 } -Popup -Async
 
 $limitsize = 10mb
 $labellimitsize = 15mb
 $window = Boots { 
    DockPanel {
       Frame `
          -Name TransitionBox -Ov global:container   `
          -MinWidth 400 -MinHeight 400 -MaxHeight 600 `
          -Content {
             StackPanel {
                Label -FontSize 42 "Loading ..."
                if($comical) {
                   Image -Source $comic.FullName -MaxWidth $comicwidth
                }
             } -'JournalEntry.Name' "Loading Screen" -Passthru
       }
    } -LastChildFill
 } -MinHeight 400 -Async -Popup -Passthru
 
 sleep 2;
 
 $jobs = @()
 ForEach($pc in $hosts) {
    Write-Verbose "Getting procs from $pc"
    $jobs += gwmi Win32_Process -ComputerName $pc -AsJob;
 }
 
 while($jobs) {
    $global:job = Wait-Job -Any $jobs 
 
    Invoke-BootsWindow $window {
 
       $name = $($job.Location -Replace "[^a-zA-Z_0-9]" -replace "(^[0-9])",'_$1')
       $global:container[0].Content = `
          $(
             Chart {
                DataSeries -LabelText $job.Location {
                   ForEach($proc in (Receive-Job $job | Sort WorkingSetSize)) {
                      if($proc.WorkingSetSize -gt $limitsize) {
                         DataPoint -YValue $proc.WorkingSetSize -LabelText $proc.Name `
                                   -LabelEnabled $($proc.WorkingSetSize -gt $labellimitsize) `
                      }
                   }
                } -RenderAs Pie -ShowInLegend:$false
             } -Watermark:$false -AnimationEnabled -Name $name -'JournalEntry.Name' $name -Passthru
          )
    }
    
    $jobs = $jobs -ne $job
    Remove-Job $job.Id
    Sleep 5
 }
`

