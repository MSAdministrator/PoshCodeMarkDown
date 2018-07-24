---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 3.3.3
Encoding: ascii
License: cc0
PoshCode ID: 705
Published Date: 2009-12-04t09
Archived Date: 2017-01-08t14
---

# start-demo - 

## Description

this is an overhaul of jeffrey snover’s original start-demo script … i’ve switched it to use readkey, which saves you some typing and makes the whole thing seem more natural when you’re demoing, (at least to me). i’ve also added a bunch of command-line options and a couple of features in the process (see the revision history in the script).

## Comments



## Usage



## TODO



## script

`read-char`

## Code

`#
 #
 ##################################################################################################
 ##
 ##
 ##################################################################################################
 ##################################################################################################
 ##
 param(
   $file=".\demo.txt", 
   [int]$command=0, 
   [System.ConsoleColor]$promptColor="Yellow", 
   [System.ConsoleColor]$commandColor="White", 
   [System.ConsoleColor]$commentColor="Green", 
   [switch]$FullAuto,
   [int]$AutoSpeed = 3,
   [switch]$NoPauseAfterExecute
 )
 
 $RawUI = $Host.UI.RawUI
 $hostWidth = $RawUI.BufferSize.Width
 
 function Read-Char() {
   $_OldColor = $RawUI.ForeGroundColor
   $RawUI.ForeGroundColor = "Red"
   $inChar=$RawUI.ReadKey("IncludeKeyUp")
   while($inChar.Character -eq 0){
     $inChar=$RawUI.ReadKey("IncludeKeyUp")
   }
   $RawUI.ForeGroundColor = $_OldColor
   return $inChar.Character
 }
 
 function Rewind($lines, $index, $steps = 1) {
    $started = $index;
    $index -= $steps;
    while(($index -ge 0) -and ($lines[$index].Trim(" `t").StartsWith("#"))){
       $index--
    }
    if( $index -lt 0 ) { $index = $started }
    return $index
 }
 
 $file = Resolve-Path $file
 while(-not(Test-Path $file)) {
   $file = Read-Host "Please enter the path of your demo script (Crtl+C to cancel)"
   $file = Resolve-Path $file
 }
 
 Clear-Host
 
 $_lines = Get-Content $file
 $_lines += "Write-Host 'The End'"
 $_starttime = [DateTime]::now
 $FullAuto = $false
 
 Write-Host -nonew -back black -fore $promptColor $(" " * $hostWidth)
 Write-Host -nonew -back black -fore $promptColor @"
 <Demo Started :: $(split-path $file -leaf)>$(' ' * ($hostWidth -(18 + $(split-path $file -leaf).Length)))
 "@
 Write-Host -nonew -back black -fore $promptColor "Press"
 Write-Host -nonew -back black -fore Red " ? "
 Write-Host -nonew -back black -fore $promptColor "for help.$(' ' * ($hostWidth -17))"
 Write-Host -nonew -back black -fore $promptColor $(" " * $hostWidth)
 
 for ($_i = $Command; $_i -lt $_lines.count; $_i++)
 {  
 	$Dur = [DateTime]::Now - $_StartTime
    $RawUI.WindowTitle = "$(if($dur.Hours -gt 0){'{0}h '})$(if($dur.Minutes -gt 0){'{1}m '}){2}s   {3}" -f 
                         $dur.Hours, $dur.Minutes, $dur.Seconds, $($_Lines[$_i])
 
 	Write-Host -nonew -fore $promptColor "[$_i]$([char]0x2265) "
 	if ($_lines[$_i].Trim(" ").StartsWith("#") -or $_lines[$_i].Trim(" ").Length -le 0) { 
 		Write-Host -fore $commentColor "$($_Lines[$_i])  "
 		continue 
 	} else {
 		Write-Host -nonew -fore $commandColor "$($_Lines[$_i])  "
 	}
 
 	if( $FullAuto ) { Start-Sleep $autoSpeed; $ch = [char]13 } else { $ch = Read-Char }
 	switch($ch)
 	{
 		"?" {
 			Write-Host -Fore $promptColor @"
 
 Running demo: $file
 (n) Next       (p) Previous
 (q) Quit       (s) Suspend 
 (t) Timecheck  (v) View $(split-path $file -leaf)
 (g) Go to line by number
 (f) Find lines by string
 (a) Auto Execute mode
 (c) Clear Screen
 "@
 		}
 			Write-Host -Fore $promptColor "<Skipping Line>"
 		}
 			Write-Host -Fore $promptColor "<Back one Line>"
 			while ($_lines[--$_i].Trim(" ").StartsWith("#")){}
 		}
 			$AutoSpeed = [int](Read-Host "Pause (seconds)")
 			$FullAuto = $true;
 			Write-Host -Fore $promptColor "<eXecute Remaining Lines>"
 		}
 			Write-Host -Fore $promptColor "<Quiting demo>"
 			$_i = $_lines.count;
 			break;
 		}
 			$lines[0..($_i-1)] | Write-Host -Fore Yellow 
 			$lines[$_i]        | Write-Host -Fore Green
 			$lines[($_i+1)..$lines.Count] | Write-Host -Fore Yellow 
 		}
 			 $dur = [DateTime]::Now - $_StartTime
        Write-Host -Fore $promptColor $(
           "{3} -- $(if($dur.Hours -gt 0){'{0}h '})$(if($dur.Minutes -gt 0){'{1}m '}){2}s" -f 
           $dur.Hours, $dur.Minutes, $dur.Seconds, ([DateTime]::Now.ToShortTimeString()))
 		}
 			Write-Host -Fore $promptColor "<Suspending demo - type 'Exit' to resume>"
 			$Host.EnterNestedPrompt()
 		}
 			$i = [int](Read-Host "line number")
 			if($i -le $_lines.Count) {
 				if($i -gt 0) {
                $_i = Rewind $_lines $_i (($_i-$i)+1)
 				} else {
 				}
 			}
 		}
 			$match = $_lines | Select-String (Read-Host "search string")
 			if($match -eq $null) {
 				Write-Host -Fore Red "Can't find a matching line"
 			} else {
 				$match | % { Write-Host -Fore $promptColor $("[{0,2}] {1}" -f ($_.LineNumber - 1), $_.Line) }
 				if($match.Count -lt 1) {
 				} else {               
 				}
 			}
 		}
       "c" { 
          Clear-Host
       }
 			Write-Host
 			trap [System.Exception] {Write-Error $_; continue;}
 			Invoke-Expression ($_lines[$_i]) | out-default
 			if(-not $NoPauseAfterExecute -and -not $FullAuto) { 
 			}
 		}
 		default
 		{
 			Write-Host -Fore Green "`nKey not recognized.  Press ? for help, or ENTER to execute the command."
 		}
 	}
 }
 $dur = [DateTime]::Now - $_StartTime
 Write-Host -Fore $promptColor $(
    "<Demo Complete -- $(if($dur.Hours -gt 0){'{0}h '})$(if($dur.Minutes -gt 0){'{1}m '}){2}s>" -f 
    $dur.Hours, $dur.Minutes, $dur.Seconds, [DateTime]::Now.ToLongTimeString())
 Write-Host -Fore $promptColor $([DateTime]::now)
 Write-Host
`

