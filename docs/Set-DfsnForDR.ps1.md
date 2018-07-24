---
Author: vcosonok
Publisher: 
Copyright: 
Email: 
Version: 5.1
Encoding: ascii
License: cc0
PoshCode ID: 5495
Published Date: 2014-10-09t09
Archived Date: 2014-10-11t09
---

# set-dfsnfordr - 

## Description

see www.cosonok.com and set-dfsnfordr.ps1.

## Comments



## Usage



## TODO



## function

`pad-r`

## Code

`#
 #
 #########################################################
 #########################################################
 
 
 #############################
 #############################
 
 
 
 
 	
 
 ##################
 ##################
 
 
 	
 
 
     }                                                                    #
 
         }                                                                    #
 
             Wr-E                                          #
 
 
 
 
 
         Columnize ($global:tableOfLinks[$r]) $x1 White ($global:tableOfLinks[$r+1]) $x2 White ($global:tableOfLinks[$r+2]) $x3 White ($global:tableOfLinks[$r+3]) $x4 White
 
 
 
 	$str = $global:targets.GetEnumerator() | Sort-Object -Property Name | Format-Table -Autosize | Out-String
 
 
 ########################
 ########################
 
         Columnize "Namespace Link" $x1 White "Target" $x2 White "State" $x3 White "PriorityClass" $x4 White
     If($global:first)   {[Void](Update-DFSUtilProperty -Match ($global:first)    -P1 "priorityclass" -P2 "set" -S1 "GlobalHigh" -Column "PriorityClass" -Display "GlobalHigh")}
     If($global:last)    {[Void](Update-DFSUtilProperty -Match ($global:last)     -P1 "priorityclass" -P2 "set" -S1 "GlobalLow"  -Column "PriorityClass" -Display "GlobalLow")}
     If($global:enabled) {[Void](Update-DFSUtilProperty -Match ($global:enabled)  -P1 "state"         -P2 "online"               -Column "State"         -Display "ONLINE")}
     If($global:disabled){[Void](Update-DFSUtilProperty -Match ($global:disabled) -P1 "state"         -P2 "offline"              -Column "State"         -Display "OFFLINE")}
 
     If ($safe -and !$ShowCommands)     {Wr-E; Wr-G "Running in safe mode, no updates will be actioned!";Wr-E}
     elseif (!$safe -and !$ShowCommands){Wr-E; Wr-C "DFSUtil property Updates being actioned!";Wr-E}
                 If($Column -eq "PriorityClass"){Columnize $namespaceLink $x1 White $target $x2 White $state $x3 White $Display $x4 Cyan}
                 If($Column -eq "State"){Columnize $namespaceLink $x1 White $target $x2 White $Display $x3 Cyan $priorityClass $x4 White}
 
 ##################
 ##################
 
 If($PSIse)                {Wr-E;Wr-R "This program cannot run in PowerShell ISE. ISE does not support ReadKey!";Wr-E;EXIT}
 If(!(Check-OSVersion 6 0)){Wr-E;Wr-R "This program requires Windows Version 6.0 (Windows 2008) or better!";Wr-E;Prompt-Keys -AnyKey;EXIT}
 If(!(Got-DFSUtil))        {Wr-E;Wr-R "This program requires access to DFSUTIL.EXE!";Wr-E;Prompt-Keys -AnyKey;EXIT}
 
 
 
 
 
 
 
     If ($reset1){$reset1 = $gotNameSpaces = $null; $nsChoice = "ALL"; $reset2 = $true}
     If ($reset2){$reset2 = $backup = $null; $reset3 = $true}
 	If ($reset3){$reset3 = $global:tableOfLinks = $global:targets = $null; $reset4 = $true}
 	If ($reset4){$reset4 = $global:first = $global:last = $global:enabled = $global:disabled = $null; $safeMode = $true}
 
     
 
 	If($backupPath) {            Wn-W "    <B>   Change backup folder. Current = ";Wr-C $backupPath}
 	else {                       Wr-R "    <B>   Select a backup folder!"}    
     If($domainServersFile){      Wn-W "    <1>   Reload Domain and Server(s) from file. Last loaded = ";Wr-C $domainServersFile}
     else{                        Wr-W "    <1>   Load Domain and Server(s) from file"}
     If($domainName){             Wn-W "    <2>   Change Domain. Currently selected = ";Wr-G $domainName}
     else{                        Wr-W "    <2>   Enter Domain Name (for Domain-based namespaces)"}
     If($servers){                Wn-W "    <3>   Change Server(s). Currently selected = ";Wr-G $servers}
     else{                        Wr-W "    <3>   Enter Server Name(s) (for Standalone namespaces)"}
     If($domainOrServer){    Wr-E;Wr-W "    <4>   Display Namespaces";$PROMPTKEYS += "4"}
     If($gotNameSpaces){          Wn-W "    <5>   Select an individual namespace/select ALL. Currently selected = ";Wr-C $nsChoice;$PROMPTKEYS += "5"}
     If($gotNameSpaces -and $backupPath){
 								 Wr-W "    <6>   Backup Namespaces";$PROMPTKEYS += "6"}
     If($backup){                 Wr-W "    <7>   Display Table of Links, Target, State and PriorityClass";$PROMPTKEYS += "7"}
     If($global:tableOfLinks){    Wr-W "    <8>   Display List of Servers with Targets in the Namespaces";$PROMPTKEYS += "8"}
     $common = "Set Server whose Targets are to be set as"                        
     If($global:targets){    Wr-E;Wn-W "    <T>   (T)oggle Mode of Operation. Current = ";Wr-C $modes[$mode];$PROMPTKEYS += "T"
         If($mode -eq 0){ $PROMPTKEYS += "F","L"
             If($global:first){   Wn-W "    <F>   $common Priority (F)irst. "; Wr-G ("Selected = " + $global:first)}
             else{                Wr-W "    <F>   $common Priority (F)irst."}
             If($global:last){    Wn-W "    <L>   $common Priority (L)ast . "; Wr-G ("Selected = " + $global:last)}
             else{                Wr-W "    <L>   $common Priority (L)ast ."}
         } else { $PROMPTKEYS += "E","D"
             If($global:enabled){ Wn-W "    <E>   $common (E)nabled . "; Wr-C ("Selected = " + $global:enabled)}
             else{                Wr-W "    <E>   $common (E)nabled ."}
             If($global:disabled){Wn-W "    <D>   $common (D)isabled. "; Wr-C ("Selected = " + $global:disabled)}
             else{                Wr-W "    <D>   $common (D)isabled."}}} 
     If($global:last -or $global:first -or $global:enabled -or $global:disabled){
         $PROMPTKEYS += "S","C","U"
                             Wr-E;Wr-C "    <C>   Display DFSUTIL (C)ommands that will be run."
         If($safeMode){           Wn-G "    <S>   Toggle (S)afe Mode On/Off. Current = "; Wr-G $safeModes[0]
                                  Wr-G "    <U>   (U)PDATE DFS!"
         } else {                 Wn-Y "    <S>   Toggle (S)afe Mode On/Off. Current = "; Wr-Y $safeModes[1]
                                  Wr-Y "    <U>   (U)PDATE DFS!"}}
                             Wr-E;Wr-W "    <X/Q> E(X)it/(Q)uit!"
                             Wr-E;Wn-G "    <<<<< Press a Key: "
 
     
     $key = Prompt-Keys $PROMPTKEYS;Wr-Y $key; Wr-E
     
     If(($key -eq "X") -or ($key -eq "Q")){EXIT}
 	If($key -eq "B"){$backupPath = Rd-W "Enter backup folder path"; $backupPath = Create-Folder $backupPath; Wr-E; $reset2 = $true}
     If($key -eq "1"){$domainServersFile = Prompt-ForDomainServersFile; $loadFile = $true; $reset1 = $true}
     If($key -eq "2"){$domainName = Prompt-ForDomain; $reset1 = $true}
     If($key -eq "3"){$servers = Prompt-ForServer; $reset1 = $true}
 		$backup = Backup-Namespaces -Domain $domainName -Server $servers -Namespace $nsChoice -Path $backupPath
 
 
 #########################
 #########################
`

