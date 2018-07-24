---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 603
Published Date: 
Archived Date: 2008-09-26t19
---

# robocopywrapper - 

## Description

powershell robocopy wrapper example code

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 #############################################################################################
 #############################################################################################
 
 
 $RoboHelp = robocopy /? | Select-String '::'
 $r = [regex]'(.*)::(.*)'
 $RoboHelpObject = $RoboHelp | select `
     @{Name='Parameter';Expression={ $r.Match( $_).groups[1].value.trim()}}, 
     @{Name='Description';Expression={ $r.Match( $_).groups[2].value.trim()}}
 
 $RoboHelpObject = $RoboHelpObject |% {$Cat = 'General'} {
     if ($_.parameter -eq '') { if ($_.Description -ne ''){
         $cat = $_.description -replace 'options :',''}
     } else {
         $_ | select @{Name='Category';Expression={$cat}},parameter,description
     }
 }
 
 
 #############################################################################################
 #############################################################################################
 
 
 robocopy 'c:\test1' c:\PowerShellRoboTest /r:2 /w:5 /s /v /np |
   Tee-Object -Variable RoboLog
 
 
 #############################################################################################
 #############################################################################################
 
 
 $null,$StartBegin,$StartEnd,$StopBegin = $RoboLog | Select-String  "----" |% {$_.linenumber}
 
 $RoboStatus = New-Object object
 
 
 $robolog[$StartBegin..$StartEnd] | % {
   Switch -regex ($_) {
     'Started :(.*)' {
       Add-Member -InputObject $RoboStatus -Name StartTime `
        -Value ([datetime]::ParseExact($matches[1].trim(),'ddd MMM dd HH:mm:ss yyyy',$null)) `
        -MemberType NoteProperty
     }
     'Source :(.*)' {
       Add-Member -InputObject $RoboStatus -Name Source `
         -Value ($matches[1].trim()) -MemberType NoteProperty
     }
     'Dest :(.*)' {
       Add-Member -InputObject $RoboStatus -Name Destination `
         -Value ($matches[1].trim()) -MemberType NoteProperty
     }    
     'Files :(.*)' {
       Add-Member -InputObject $RoboStatus -Name FileName `
         -Value ($matches[1].trim()) -MemberType NoteProperty
     }
     'Options :(.*)' {
       Add-Member -InputObject $RoboStatus -Name Options `
         -Value ($matches[1].trim()) -MemberType NoteProperty
     }
   }
 }
 
 
 $robolog[$StopBegin..( $RoboLog.Count  -1)] |% {
   Switch -regex ($_) {
 
     'Ended :(.*)' {
         Add-Member -InputObject $RoboStatus -Name StopTime `
           -Value ([datetime]::ParseExact($matches[1].trim(),'ddd MMM dd HH:mm:ss yyyy',$null))`
           -MemberType NoteProperty
     }
 
     'Speed :(.*) Bytes' {
         Add-Member -InputObject $RoboStatus -Name BytesSecond `
           -Value ($matches[1].trim()) -MemberType NoteProperty
     }
 
     'Speed :(.*)MegaBytes' {
         Add-Member -InputObject $RoboStatus -Name MegaBytesMinute `
           -Value ($matches[1].trim()) -MemberType NoteProperty
     }    
 
     '(Total.*)' {
       $cols = $_.Split() |? {$_}
     }
 
     'Dirs :(.*)' {
       $fields = $matches[1].Split() |? {$_}
       $dirs = new-object object
       0..5 |% {
           Add-Member -InputObject $Dirs -Name $cols[$_] -Value $fields[$_] -MemberType NoteProperty
           Add-Member -InputObject $Dirs -Name 'toString' -MemberType ScriptMethod `
             -Value {[string]::Join(" ",($this.psobject.Properties |
               % {"$($_.name):$($_.value)"}))} -force
       }
       Add-Member -InputObject $RoboStatus -Name Directories -Value $dirs -MemberType NoteProperty
     }
 
     'Files :(.*)' {
       $fields = $matches[1].Split() |? {$_}
       $Files = new-object object
       0..5 |% {
           Add-Member -InputObject $Files -Name $cols[$_] -Value $fields[$_] -MemberType NoteProperty
           Add-Member -InputObject $Files -Name 'toString' -MemberType ScriptMethod -Value `
             {[string]::Join(" ",($this.psobject.Properties |% {"$($_.name):$($_.value)"}))} -force
       }
       Add-Member -InputObject $RoboStatus -Name files -Value $files -MemberType NoteProperty
     }
 
     'Bytes :(.*)' {
       $fields = $matches[1].Split() |? {$_}
       $fields = $fields |% {$new=@();$i = 0 } {
           if ($_ -match '\d') {$new += $_;$i++} else {$new[$i-1] = ([double]$new[$i-1]) * "1${_}B" }
       }{$new}
 
       $Bytes = new-object object
       0..5 |% {
           Add-Member -InputObject $Bytes -Name $cols[$_] `
             -Value $fields[$_] -MemberType NoteProperty
           Add-Member -InputObject $Bytes -Name 'toString' -MemberType ScriptMethod `
             -Value {[string]::Join(" ",($this.psobject.Properties |
             % {"$($_.name):$($_.value)"}))} -force
       }
       Add-Member -InputObject $RoboStatus -Name bytes -Value $bytes -MemberType NoteProperty
     }
   }
 }
 
 
 $re = New-Object regex('(.*)\s{2}([\d\.]*\s{0,1}\w{0,1})\s(.*)')
 $RoboDetails = $robolog[($StartEnd +1)..($stopbegin -3)] |? {$_.StartsWith([char]9)} | select `
     @{Name='Action';Expression={$re.Match($_) |% {$_.groups[1].value.trim()}}},
     @{Name='Size';Expression={$re.Match($_) |% {$_.groups[2] |% {$_.value.trim()}}}},
     @{Name='Directory';Expression={if(!($re.Match($_) |% {$_.groups[1].value.trim()})){
       '-';$Script:dir = $re.Match($_) |% {$_.groups[3] |
       % {$_.value.trim()}} }else {$script:dir}}},
     @{Name='Name';Expression={$re.Match($_) |% {$_.groups[3] |% {$_.value.trim()}}}}
 
 
 0..($RoboDetails.count -1) |% {
   if ($Robodetails[$_].Directory -eq '-') {
     $Robodetails[$_].Action = 'Directory'
     $Robodetails[$_].Directory = split-path $Robodetails[$_].Name
   }
   if ($Robodetails[$_].size -match '[mg]') {
     $Robodetails[$_].size = [double]($roboDetails[$_].size.trim('mg ')) * 1mb
   }
 }
 
   -Value {"Details : " + $this.count} -force
 Add-Member -InputObject $RoboStatus -Name Details `
   -Value $RoboDetails -MemberType NoteProperty
 
 
 $reWarning = New-Object regex('(.*)(ERROR.*)(\(.*\))(.*)\n(.*)')
 $roboWarnings = $reWarning.matches(($robolog | out-string)) | Select `
     @{Name='Time';Expression={[datetime]$_.groups[1].value.trim()}},
     @{Name='Error';Expression={$_.groups[2].value.trim()}},
     @{Name='Code';Expression={$_.groups[3].value.trim()}},
     @{Name='Message';Expression={$_.groups[5].value.trim()}},
     @{Name='Info';Expression={$_.groups[4].value.trim()}} 
 
   -MemberType ScriptMethod -Value {"Details : " + $this.count} -force
 Add-Member -InputObject $RoboStatus -Name Warnings `
   -Value $roboWarnings -MemberType NoteProperty
 
 $reErrors = New-Object regex('\) (.*)\n(.*)\nERROR:(.*)')
 $roboErrors = $reErrors.matches(($robolog |? {$_}| out-string)) | Select `
     @{Name='Error';Expression={$_.groups[3].value.trim()}},
     @{Name='Message';Expression={$_.groups[2].value.trim()}},
     @{Name='Info';Expression={$_.groups[1].value.trim()}}
 
   -MemberType ScriptMethod -Value {"Details : " + $this.count} -force
 Add-Member -InputObject $RoboStatus -Name Errors `
   -Value $RoboErrors -MemberType NoteProperty
 
 
 #############################################################################################
 #############################################################################################
 
 
 
 $RoboStatus
 
 
 "Time elapsed : $($RoboStatus.StopTime - $RoboStatus.StartTime)"
 
 
 $RoboStatus.Options.split()[1..100] |
   % { $par = $_ ;$RoboHelpObject |? {$_.parameter -eq $par} } | ft -a
 
 
 $RoboStatus.files
 $RoboStatus.Directories
 
 
 $RoboStatus.Details | group action
 
 
 $RoboStatus.Errors | fl
 
 $RoboStatus.Warnings
 
 
 $RoboStatus.Warnings |group info | ft count,Name -a
 
 
 $RoboStatus.Warnings | 
   select *, @{name='Failed';e={($RoboStatus.errors |% {$_.info}) -contains $_.info}}
 
 
 $RoboStatus.details |? {$_.action} | select *,
   @{name='Failed';e={$d = $_;($RoboStatus.errors |
   % {$_.info}) -match '\\'+([regex]::escape($d.name))}} |? {$_.failed} |
   group action,directory,name | ft -a name,count
 
 
 $RoboStatus.Errors | 
   select *,@{name='Warnings';e={$e = $_;($robostatus.warnings |? {$_.info -eq $e.info}).count}}
 
 
 $RoboStatus.Errors | 
   select *,@{name='Warnings';e={$e = $_;($robostatus.warnings |? {$_.info -eq $e.info})}} |% {
     $_ | fl error,Info ;$_.warnings | sort -u info,message | ft [tecm]* -a  
 }
`

