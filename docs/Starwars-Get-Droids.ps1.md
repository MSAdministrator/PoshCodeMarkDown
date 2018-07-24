---
Author: james vahanian
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 4760
Published Date: 2014-01-02t08
Archived Date: 2014-01-04t21
---

# starwars get-droids - 

## Description

inspired from http

## Comments



## Usage



## TODO



## script

`get-droids`

## Code

`#
 #
 
 $jedi = @()
     $jedi1 = New-Object �TypeName PSObject
     $jedi1 | Add-Member -MemberType NoteProperty �Name Name �Value "Obiwan Kenobe"
     $jedi1 | Add-Member �MemberType NoteProperty �Name NickName �Value "Old Ben"
     $jedi1 | Add-Member �MemberType NoteProperty �Name Status �Value "Retired"
     $jedi1 | Add-Member -MemberType ScriptProperty -Name GetName -value {$this.Name}
 
     $jedi2 = New-Object �TypeName PSObject
     $jedi2 | Add-Member -MemberType NoteProperty �Name Name �Value "Master Yoda"
     $jedi2 | Add-Member �MemberType NoteProperty �Name NickName �Value "Yoda"
     $jedi2 | Add-Member �MemberType NoteProperty �Name Status �Value "Retired"
     $jedi2 | Add-Member -MemberType ScriptProperty -Name GetName -value {$this.Name}
 $jedi += $jedi1, $jedi2
 
 $droids = @()
     $droid1 = New-Object �TypeName PSObject
     $droid1 | Add-Member -MemberType NoteProperty �Name Name �Value "R2D2"
     $droid1 | Add-Member �MemberType NoteProperty �Name Model �Value "Astro Droid"
     $droid1 | Add-Member �MemberType NoteProperty �Name Function �Value "Help bring in the Moisture Harvest For Master Luke"
     $droid1 | Add-Member �MemberType NoteProperty �Name RealFunction �Value "Get Deathstar plans to $ObiWan"
     $droid1 | Add-Member �MemberType NoteProperty �Name PreviousOwner �Value "Leia Morgana"
     $droid1 | Add-Member -MemberType ScriptProperty -Name GetName -value {$this.Name}
 
     $droid2 = New-Object �TypeName PSObject
     $droid2 | Add-Member -MemberType NoteProperty �Name Name �Value "C3PO"
     $droid2 | Add-Member �MemberType NoteProperty �Name Model �Value "Protocol Droid - Human Cyborg Relations"
     $droid2 | Add-Member �MemberType NoteProperty �Name Function �Value "General annoyance and Comedic Relief, oh and communicate with Moisture Vaporators"
     $droid2 | Add-Member �MemberType NoteProperty �Name RealFunction �Value "Assist R2D2 with assigned mission"
     $droid2 | Add-Member �MemberType NoteProperty �Name PreviousOwner �Value "Leia Morgana"
     $droid2 | Add-Member -MemberType ScriptProperty -Name GetName -value {$this.Name}
 $droids += $droid1, $droid2
 $ObiWan = ($jedi.GetName -eq 'Obiwan Kenobe')
 
 $defaultProperties = @(�Name�,'Model','Function')
 $defaultDisplayPropertySet = New-Object System.Management.Automation.PSPropertySet(�DefaultDisplayPropertySet�,[string[]]$defaultProperties)
 $PSStandardMembers = [System.Management.Automation.PSMemberInfo[]]@($defaultDisplayPropertySet)
 $droid1 | Add-Member MemberSet PSStandardMembers $PSStandardMembers
 $droid2 | Add-Member MemberSet PSStandardMembers $PSStandardMembers
 
 Function Get-Droids {
     PARAM (
     [Parameter(Mandatory=$True)]
     [PSCustomObject]$PotentialDroids,
     [Parameter(Mandatory=$False)]
     [bool]$UseForce
     )
     foreach($droid in $PotentialDroids)
     {
         if($droid.Function -match "Deathstar")
         {
             Write-Host "Detain Droids for questioning!" -ForegroundColor Red
             return $True
         }
         elseif ($UseForce)
         {
            $droid |select Name, RealFunction |ft -AutoSize -force
         }
         else
         {
             return $False
         }
     }   
 }
 
 $StormTrooperSearch = Get-Droids -PotentialDroids $droids -UseForce $False
 If($StormTrooperSearch -eq $False){ Write-Host "These's aren't the Droids we're looking for, Move Along" -ForegroundColor Green}
 
 $JediSearch = Get-Droids -PotentialDroids $droids -UseForce $True
 If($JediSearch -eq $False)
 { 
     Write-Host "Who the heck are these Droids?!?" -ForegroundColor Red
 }
 else
 {
     $JediSearch
 }
`

