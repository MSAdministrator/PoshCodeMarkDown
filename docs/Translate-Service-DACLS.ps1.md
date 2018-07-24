---
Author: claus t nielsen
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2233
Published Date: 
Archived Date: 2010-09-22t05
---

# translate service dacls - 

## Description

script to translate service dacl’s into hrl (human readable language)

## Comments



## Usage



## TODO



## function

`get-servicedacl`

## Code

`#
 #
 Function Get-ServiceDACL {
 [CmdletBinding()]
     param(
         [Parameter(Mandatory=$true,Position=0,ValueFromPipeline=$true)]
         [String]$Servicename,
         [Parameter(Mandatory=$false,Position=1)]
         [String]$Computername= ".")
 	
 
 $sddl
 $parts =  $sddl -split(":")
 #$parts.Length
 $i = 0
 Write-Host "Getting Service DACL for $ServiceName on $Computername"
 While ($i -lt $parts.length) {
 
 $part = $parts[$i]
 
 Switch ($part) {
 "D" { $i++; Parse-DACL $parts[$i] }
 }
 $i++
 }
 $sddl = ""
 }
 
 
 Function Parse-DACL {
 Param([String]$SDDLIN)
 
 [Array]$sddls = ($SDDLIN).split('(')
 Foreach ($SDDLI in $sddls) {
 #($SDDLI).replace(')';'') 
 #$SDDLI
 $tokens = (($SDDLI).replace(')','')).split(";")
 If ($tokens[5]) {
 If ($tokens[5].length -gt 3) {
 [wmi]$obj = 'Win32_SID.SID="{0}"' -f $($tokens[5])
     $encoded = [System.Convert]::ToBase64String($obj.BinaryRepresentation)
     $obj | Add-Member -MemberType NoteProperty -Name base64_sid -Value $encoded
 	Write-Host "$($obj.ReferencedDomainName)\$($obj.AccountName)" -ForegroundColor red
 }
 Else {
 Write-Host "$($Trustees.get_item($tokens[5]))" -ForegroundColor red
 }
  "   " + $AceType.get_item($tokens[0])
  [regex]::split($tokens[2], '(.{2})') | % {Write-host "      $($PermissionType.get_item($_)) `n" -NoNewline}
  }
 }
 }
 
 $AceType = @{"A" = "ACCESS ALLOWED";
 "D" = "ACCESS DENIED";
 "OA" = "OBJECT ACCESS ALLOWED: ONLY APPLIES TO A SUBSET OF THE OBJECT(S).";
 "OD" = "OBJECT ACCESS DENIED: ONLY APPLIES TO A SUBSET OF THE OBJECT(S).";
 "AU" = "SYSTEM AUDIT";
 "AL" = "SYSTEM ALARM";
 "OU" = "OBJECT SYSTEM AUDIT";
 "OL" = "OBJECT SYSTEM ALARM";
 "ML" = "MANDATORY LABEL"}
 
 $AceFlags = @{
 "CI" = "CONTAINER INHERIT: Child objects that are containers, such as directories, inherit the ACE as an explicit ACE.";
 "OI" = "OBJECT INHERIT: Child objects that are not containers inherit the ACE as an explicit ACE.";
 "NP" = "NO PROPAGATE: ONLY IMMEDIATE CHILDREN INHERIT THIS ACE.";
 "IO" = "INHERITANCE ONLY: ACE DOESN'T APPLY TO THIS OBJECT; BUT MAY AFFECT CHILDREN VIA INHERITANCE.";
 "ID" = "ACE IS INHERITED";
 "SA" = "SUCCESSFUL ACCESS AUDIT";
 "FA" = "FAILED ACCESS AUDIT"
 }
 
 $PermissionType = @{
 "CC" = "Query Conf";
 "DC" = "Change Conf";
 "LC" = "QueryStat";
 "SW" =	"EnumDeps";
 "RP" =	"Start";
 "WP" =	"Stop";
 "DT" =	"Pause";
 "LO" =	"Interrogate";
 "CR" =	"UserDefined";
 "GA" =	"Generic All";
 "GX" =	"Generic Execute";
 "GW" =	"Generic Write";
 "GR" =	"Generic Read";
 "SD" =	"Standard Delete";
 "RC" =	"Read Control";
 "WD" =  "Write DAC";
 "WO" =	"Write Owner"
 }
 
 
 $Trustees = @{
 "AO" = "Account operators";
 "RU" = "Alias to allow previous Windows 2000";
 "AN" = "Anonymous logon";
 "AU" = "Authenticated users";
 "BA" = "Built-in administrators";
 "BG" = "Built-in guests";
 "BO" = "Backup operators";
 "BU" = "Built-in users";
 "CA" = "Certificate server administrators";
 "CG" = "Creator group";
 "CO" = "Creator owner";
 "DA" = "Domain administrators";
 "DC" = "Domain computers";
 "DD" = "Domain controllers";
 "DG" = "Domain guests";
 "DU" = "Domain users";
 "EA" = "Enterprise administrators";
 "ED" = "Enterprise domain controllers";
 "WD" = "Everyone";
 "PA" = "Group Policy administrators";
 "IU" = "Interactively logged-on user";
 "LA" = "Local administrator";
 "LG" = "Local guest";
 "LS" = "Local service account";
 "SY" = "Local system";
 "NU" = "Network logon user";
 "NO" = "Network configuration operators";
 "NS" = "Network service account";
 "PO" = "Printer operators";
 "PS" = "Personal self";
 "PU" = "Power users";
 "RS" = "RAS servers group";
 "RD" = "Terminal server users";
 "RE" = "Replicator";
 "RC" = "Restricted code";
 "SA" = "Schema administrators";
 "SO" = "Server operators";
 "SU" = "Service logon user"
 }
 
 
 
 Get-ServiceDACL winrm RemoteServer
`

