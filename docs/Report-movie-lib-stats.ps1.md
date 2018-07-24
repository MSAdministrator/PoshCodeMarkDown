---
Author: james gentile
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3171
Published Date: 2012-01-17t13
Archived Date: 2017-05-22t04
---

# report movie lib. stats - 

## Description

this is a script that reports movie library statistics, it’s hard coded to work on my two lan computers so you will have to do some hacking if you want to get it to work.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function getmoviestat ($drives) { 
 
 $drives=$drives | %{if ($_.systemname -eq $env:computername) {add-member -inputobject $_ -name BRs -membertype noteproperty -value ( gci -ea 0 "$($_.name)\media\BR movies\*" -include *.mkv,*.m2ts  ) -passthru } else {add-member -inputobject $_ -name BRs -membertype noteproperty -value ( invoke-command -session $s -scriptblock { param ($name) gci -ea 0 "$name\media\BR movies\*" -include *.mkv,*.m2ts } -argumentlist $_.name ) -passthru}}
 
 $drives=$drives | %{if ($_.systemname -eq $env:computername) { add-member -inputobject $_ -name SDs -membertype noteproperty -value ( gci -ea 0 "$($_.name)\media\SD movies\*" -include *.mkv,*.avi  ) -passthru } else {add-member -inputobject $_ -name SDs -membertype noteproperty -value ( invoke-command -session $s -scriptblock { param ($name) gci -ea 0 "$name\media\SD movies\*" -include *.mkv,*.avi } -argumentlist $_.name) -passthru}}
 
 $drives=$drives | %{ add-member -inputobject $_ -name MVs -membertype scriptproperty -value { $this.SDs + $this.BRs }  -passthru }
 
 $drives=$drives | %{ add-member -inputobject $_ -name BRsize -membertype scriptproperty -value { $n=0;foreach($mv in $this.BRs){ $n+=$mv.length }; [math]::round($n/1gb,0)}  -passthru }
 
 $drives=$drives | %{ add-member -inputobject $_ -name SDsize -membertype scriptproperty -value { $n=0;foreach($mv in $this.SDs){ $n+=$mv.length }; [math]::round($n/1gb,0)}  -passthru }
 
 $drives=$drives | %{ add-member -inputobject $_ -name MVsize -membertype scriptproperty -value { $this.BRsize + $this.SDsize }  -passthru }
 
 }
 
 function totalstats ($drives,$systemname="All") {
     $drivesobject = new-object psobject -property @{
         systemname=$systemname
         name="All"
         capacity=  $drives | %{$s=0}{$s+=$_.capacity}{$s}
         freespace= $drives | %{$s=0}{$s+=$_.freespace}{$s}
         SDs=       $drives | %{$movs=@()}{if ($_.SDs.count -ne 0 -and $_.SDs -ne $null){$movs+=$_.SDs}}{$movs}
         BRs=       $drives | %{$movs=@()}{if ($_.BRs.count -ne 0 -and $_.BRs -ne $null){$movs+=$_.BRs}}{$movs}
         MVs=       $drives | %{$movs=@()}{if ($_.MVs.count -ne 0 -and $_.MVs -ne $null){$movs+=$_.MVs}}{$movs}
         SDsize=    $drives | %{$s=0}{$s+=$_.SDsize}{$s}
         BRsize=    $drives | %{$s=0}{$s+=$_.BRsize}{$s}
         MVSize=    $drives | %{$s=0}{$s+=$_.MVsize}{$s}
     }
         return $drivesobject
 }
 
 function displaystats ($drives) {
 $drives | ft -a systemname, name, @{Label="Size";Expression={[math]::round($_.capacity/1GB,0)}}, @{Label="Used"; Expression={[math]::round(($_.capacity-$_.freespace)/1GB,0)}},@{Label="free"; Expression={[math]::round($_.freespace/1GB,0)}}, @{Label="SDs";Expression={if ($_.SDs.count) {$_.SDs.count} else {0}}}, @{Label="BRs";Expression={ if ($_.BRs.count) {$_.BRs.count} else {0}}}, @{Label="Movs";Expression={ if ($_.MVs.count) {$_.MVs.count} else {0}}}, @{label="SD SZ";Expression={$_.SDsize}}, @{Label="BR SZ"; Expression={$_.BRsize}}, @{Label="Mov SZ"; Expression={$_.MVsize}}, @{Label="Other SZ"; Expression={[math]::round((($_.capacity/1GB)-($_.freespace/1GB))-$_.MVSize,0)}}, @{Label="<SDs>"; Expression={if ($_.SDs.count -ne 0 -and $_.SDs.count -ne $null) {[math]::round($_.SDsize/$_.SDs.count,0) } else {"?"}};align="right"}, @{Label="<BRs>"; Expression={if($_.BRsize -ne 0 -and $_.BRsize -ne $null){$global:BRave=[math]::round($_.BRsize/$_.BRs.count,0);$BRave } else {$global:BRave=0;"?"}};align="right"}, @{Label="<Movs>"; Expression={if ($_.MVsize -ne 0 -and $_.MVsize -ne $null) {[math]::round($_.MVsize/$_.MVs.count,0)} else {"?"}};align="right"}, @{Label="BR slots";Expression={ if ($BRAve -ne 0 -and $BRave -ne $null) {[math]::round($_.capacity/1GB/$BRAve)} else {"?"}}; Align="Right"},@{label="BRs Avail";Expression={if ($BRAve -eq 0 -or $BRave -eq $null) {"?"} else {[math]::round(($_.freespace/1gb)/$BRAve,0)}};Align="Right" }
 }
 
 $cn = "QuadBox"
 
 if ($env:computername -eq $cn) {$cn = "SixShooter"}
 
 $s = new-pssession -ea 0 -computername $cn -sessionOption (new-pssessionoption -noCompression)
 
 $localdrives=@(gwmi -class win32_volume | ?{$_.filesystem -eq "ntfs"} | ?{$_.name -notlike "\\*"}|?{test-path "$($_.name)\media\BR movies"}|sort driveletter)
 
 getmoviestat $localdrives
 
 $remotedrives=@()
 
 if ($s -ne $null) {$remotedrives=@(invoke-command -session $s -scriptblock {gwmi -class win32_volume | ?{$_.filesystem -eq "ntfs"} | ?{$_.name -notlike "\\*"}|?{test-path "$($_.name)\media\BR movies"}|sort driveletter}); getmoviestat $remotedrives }
 
 $localall=totalstats $localdrives $localdrives[0].systemname
 $remoteall=totalstats $remotedrives $remotedrives[0].systemname
 $all=totalstats ($remotedrives+$localdrives)
 
 displaystats ($localdrives+$localall+$remotedrives+$remoteall+$all)
 
 
 if ($s -ne $null) {remove-pssession $S}
 
 write-host "Press Any Key to continue..." -n; $host.ui.rawui.readkey() | out-null; write-host ""
`

