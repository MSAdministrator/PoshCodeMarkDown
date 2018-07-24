---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3818
Published Date: 
Archived Date: 2012-12-16t04
---

#  - 

## Description

killing processes in disconnected terminal service sessions

## Comments



## Usage



## TODO



## script

`get-sessions`

## Code

`#
 #
 [CmdletBinding(SupportsShouldProcess=$true, ConfirmImpact='Medium')]
 param (
 )
 
 function Get-Sessions
 {
 
 
     <#
     http://support.microsoft.com/kb/186592
     Active. The session is connected and active.
     Conn.   The session is connected. No user is logged on.
     ConnQ.  The session is in the process of connecting. If this state
             continues, it indicates a problem with the connection.
     Shadow. The session is shadowing another session.
     Listen. The session is ready to accept a client connection.
     Disc.   The session is disconnected.
     Idle.   The session is initialized.
     Down.   The session is down, indicating the session failed to initialize correctly.
     Init.   The session is initializing.
     #>
 
     $c = query session 2>&1 | where {$_.gettype().equals([string]) }
 
     $starters = New-Object psobject -Property @{"SessionName" = 0; "Username" = 0; "ID" = 0; "State" = 0; "Type" = 0; "Device" = 0;};
     
     foreach($line in $c) {
          try {
              if($line.trim().substring(0, $line.trim().indexof(" ")) -eq "SESSIONNAME") {
                 $starters.Username = $line.indexof("USERNAME");
                 $starters.ID = $line.indexof("ID");
                 $starters.State = $line.indexof("STATE");
                 $starters.Type = $line.indexof("TYPE");
                 $starters.Device = $line.indexof("DEVICE");
                 continue;
             }
           
             New-Object psobject -Property @{
                 "SessionName" = $line.trim().substring(0, $line.trim().indexof(" ")).trim(">")
                 ;"Username" = $line.Substring($starters.Username, $line.IndexOf(" ", $starters.Username) - $starters.Username)
                 ;"ID" = $line.Substring($line.IndexOf(" ", $starters.Username), $starters.ID - $line.IndexOf(" ", $starters.Username) + 2).trim()
                 ;"State" = $line.Substring($starters.State, $line.IndexOf(" ", $starters.State)-$starters.State).trim()
                 ;"Type" = $line.Substring($starters.Type, $starters.Device - $starters.Type).trim()
                 ;"Device" = $line.Substring($starters.Device).trim()
             }
         } catch {
             throw $_;
             #$e = $_;
         }
     }
 }
 
 $IncludedSessions = Get-Sessions `
                         | Where { $_.State -match $IncludeStates } `
                         | Select -ExpandProperty ID
 
 $SessionProcesses = $IncludedSessions `
     | % { $id = $_;
           Get-Process `
             | Where { $_.SessionID -eq $id -and $_.Name -match $KillProcesses } }
 
 if ($SessionProcesses -ne $null) {
     $SessionProcesses | Stop-Process
 }
`

