---
Author: kenneth c mazie
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: utf-8
License: cc0
PoshCode ID: 2564
Published Date: 2012-03-15t12
Archived Date: 2015-01-27t02
---

# array-randomizer.ps1 - 

## Description

originally written to regenerate the init.txt file for server based halo 1 pc multiplayer games.  it may however be easily edited and used to randomize any array or group of arrays.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #======================================================================================
 #=======================================================================================
 
 Clear-Host 
 
 #--[ Function to shuffle an array ]-------------------------------
 function Shuffle
 {
  param([Array] $a)
  $rnd=(new-object System.Random)
  for($i=0;$i -lt $a.Length;$i+=1){
   $newpos=$i + $rnd.Next($a.Length - $i); 
   $tmp=$a[$i]; 
   $a[$i]=$a[$newpos]; 
   $a[$newpos]=$tmp 
  } 
  return $a
 }
 
 #--[ Check for, and delete any existing init file ]-----------------
 if ($env:PROCESSOR_ARCHITECTURE -eq "AMD64"){$path = "C:\Program Files (x86)\Microsoft Games\Halo\init.txt"}
 else
 {$path = "C:\Program Files\Microsoft Games\Halo\init.txt"}
 if(!(Test-Path -Path $path))
   {new-item -Path $path â��itemtype file}
 else
   {remove-Item -Path $path}
   
 #--[ Please place any file header items here ]-----------------------  
 $arrHeader =        
 "sv_name halo",
 "sv_public false",
 "sv_maxplayers 16",
 "sv_mapcycle_timeout 6",
 "sv_password halo"
 #--[ Place game types here ]--------------------------------
 $arrGameType = 
 "classic_slayer",
 "classic_phantoms",
 "classic_elimination",
 "classic_juggernaut",
 "classic_snipers",
 "classic_rockets",
 "classic_invasion"
 #--[ Place your list of maps here ]--------------------------
 $arrMapList = 
 "sv_mapcycle_add dangercanyon",
 "sv_mapcycle_add gephyrophobia",
 "sv_mapcycle_add deathisland",
 "sv_mapcycle_add bloodgulch",
 "sv_mapcycle_add beavercreek",
 "sv_mapcycle_add boardingaction",
 "sv_mapcycle_add carousel",
 "sv_mapcycle_add chillout",
 "sv_mapcycle_add damnation",
 "sv_mapcycle_add hangemhigh",
 "sv_mapcycle_add icefields",
 "sv_mapcycle_add longest",
 "sv_mapcycle_add infinity",
 "sv_mapcycle_add prisoner",
 "sv_mapcycle_add putput",
 "sv_mapcycle_add ratrace",
 "sv_mapcycle_add sidewinder",
 "sv_mapcycle_add timberland",
 "sv_mapcycle_add wizard" 
 
 #--[ Randomize the main array ]-------------------------------
 shuffle $arrMapList
 
 #--[ Generate the new init file ]-----------------------------
 add-content -path $Path -value $arrHeader
 foreach ($Map in $arrMapList ) { Add-content -path $Path -value ($Map  + " " + $arrGameType[(get-random -min 0 -max ($arrGameType.length))])}
 Add-content -path $Path -value "sv_mapcycle_begin"
 
 #--[ Launch the game ]----------------------------------------
 if ($env:PROCESSOR_ARCHITECTURE -eq "AMD64"){(new-object -com shell.application).ShellExecute("C:\Program Files (x86)\Microsoft Games\Halo\haloded108.exe")}
 else
 {(new-object -com shell.application).ShellExecute("C:\Program Files\Microsoft Games\Halo\haloded108.exe")}
`

