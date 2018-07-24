---
Author: johnsinnes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 6086
Published Date: 2016-11-10t18
Archived Date: 2016-09-09t01
---

# pshell tiny videogame - 

## Description

this code makes a tiny game; you must dodge all digits that will go appearing in the screen. use the left-right keys to move yourself. when the game starts and asks you to press intro to start you can put some commands;

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Set-psdebug -strict
 [int]$op_pos_1=0
 [int]$op_pos_2=0
 [int]$op_pos_3=0
 [int]$key_aux=0
 [int]$time=0
 [int]$p_x=5
 [int]$range_factor=50
 [decimal]$delay_total=300
 [string]$pause=""
 [string]$key="0000000000000000000000000000000000000"
 [string]$keyboard_true=""
 [string]$keyboard_vkey=""
 [string]$grid=""
 [string]$whatkey=""
 [string]$g_time="|{0}" -f $time
                  #"||||||||||||||||||||||||||||||"
 [string]$grid_op=""
 
 [string]$o_a=" "
 [string]$o_b=" "
 [string]$o_c=" "
 [string]$o_d=" "
 [string]$o_e=" "
 [string]$o_f=" "
 [string]$o_g=" "
 [string]$o_h=" "
 [string]$o_i=" "
 [string]$o_j=" "
 
 [string]$g_a=" "
 [string]$g_b=" "
 [string]$g_c=" "
 [string]$g_d=" "
 [string]$g_e=" "
 [string]$g_f=" "
 [string]$g_g=" "
 [string]$g_h=" "
 [string]$g_i=" "
 [string]$g_j=" "
 
 [string]$c_a=" "
 [string]$c_b=" "
 [string]$c_c=" "
 [string]$c_d=" "
 [string]$c_e=" "
 [string]$c_f=" "
 [string]$c_g=" "
 [string]$c_h=" "
 [string]$c_i=" "
 [string]$c_j=" "
 
 [bool]$g_cont=$false
 [bool]$g_end=$false
 [bool]$gen=$true
 [bool]$collision=$false
 [bool]$walls_cfg=$false
 [bool]$god_mode=$false
 
 cls
 write-host "~" -NoNewLine -ForegroundColor White
 write-host "P" -NoNewline -ForegroundColor Cyan
 write-host "o" -NoNewline -ForegroundColor Green
 write-host "w" -NoNewline -ForegroundColor Yellow
 write-host "e" -NoNewline -ForegroundColor Red
 write-host "r" -NoNewline -ForegroundColor Magenta
 write-host " " -NoNewline -ForegroundColor White
 write-host "S" -NoNewline -ForegroundColor Cyan
 write-host "h" -NoNewline -ForegroundColor Green
 write-host "i" -NoNewline -ForegroundColor Yellow
 write-host "t" -NoNewline -ForegroundColor Red
 write-host "G" -NoNewline -ForegroundColor Magenta
 write-host "a" -NoNewline -ForegroundColor White
 write-host "m" -NoNewline -ForegroundColor Cyan
 write-host "e" -NoNewline -ForegroundColor Green
 write-host "0" -NoNewline -ForegroundColor Yellow
 write-host "." -NoNewline -ForegroundColor Red
 write-host "1" -NoNewline -ForegroundColor Magenta
 write-host "V" -NoNewline -ForegroundColor White
 
 write-host " "
 write-host "Press INTRO to start game!" -ForegroundColor Cyan
 $pause=read-host 
 if ($pause -eq "god_mode enable")
     {
     $god_mode = $true
     write-host "god_mode enabled!" -ForegroundColor White -BackgroundColor Black
     }elseif ($pause -eq "walls enable")
         {
         $walls_cfg = $true
         write-host "walls enabled!" -ForegroundColor White -BackgroundColor Black
         }elseif ($pause -eq "set help")
             {
             write-host "Set 5 : O  -X^" -ForegroundColor White -BackgroundColor Black
             $pause=read-host
             }
     if ($pause -eq "set 1")
         {
         }elseif ($pause -eq "set 2")
             {
             }elseif ($pause -eq "set 3")
                 {
                 $e_char="·"
                 }elseif ($pause -eq "set 4")
                     {
                     $e_char="·"
                     }elseif ($pause -eq "set 5")
                         {
                         $e_char="^"
                         $p_char="O"
                         $o_char="X"
                         write-host "Set 5 : O  -X^ Enabled!" -ForegroundColor White -BackgroundColor Black
                         }
 
 
 write-host $g_header -ForegroundColor DarkGreen -BackgroundColor white 
 while ($g_end -lt $true)
 {
 if ($Host.UI.RawUI.KeyAvailable) {
     $key = $Host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown,IncludeKeyUp")
 }
 $keyboard_event=$key.Substring(0,2)
 $key_aux=$key.Length
 $keyboard_true=$key[$key_aux-4]
 if (($keyboard_event -eq "37") -and ($keyboard_true -eq "T"))
 {
     $whatkey="left"
     $keyboard_true="F"
     }elseif (($keyboard_event -eq "39") -and ($keyboard_true -eq "T"))
     {
         $whatkey="right"
         $keyboard_true="F"
     }
     else
     {
     $whatkey="no"
     $keyboard_true="X"
     $keyboard_event="69"
     }
 
 
 
 
 
 
 if ($gen -eq 1)
     {
     if ($whatkey -eq "right")
         {
         $p_x++
         if ($p_x -eq 10)
             {
             $p_x=9
             }
         }
     if ($whatkey -eq "left")
         {
         $p_x--
         if ($p_x -eq -1)
             {
             $p_x=0
             }
         }
 if ($p_x -eq 0)
     {
     $g_a=$p_char
     $g_b=" "
     $g_c=" "
     $g_d=" "
     $g_e=" "
     $g_f=" "
     $g_g=" "
     $g_h=" "
     $g_i=" "
     $g_j=" "
     }
 if ($p_x -eq 1)
     {
     $g_a=" "
     $g_b=$p_char
     $g_c=" "
     $g_d=" "
     $g_e=" "
     $g_f=" "
     $g_g=" "
     $g_h=" "
     $g_i=" "
     $g_j=" "
     }
 if ($p_x -eq 2)
     {
     $g_a=" "
     $g_b=" "
     $g_c=$p_char
     $g_d=" "
     $g_e=" "
     $g_f=" "
     $g_g=" "
     $g_h=" "
     $g_i=" "
     $g_j=" "
     }
 if ($p_x -eq 3)
     {
     $g_a=" "
     $g_b=" "
     $g_c=" "
     $g_d=$p_char
     $g_e=" "
     $g_f=" "
     $g_g=" "
     $g_h=" "
     $g_i=" "
     $g_j=" "
     }
 if ($p_x -eq 4)
     {
     $g_a=" "
     $g_b=" "
     $g_c=" "
     $g_d=" "
     $g_e=$p_char
     $g_f=" "
     $g_g=" "
     $g_h=" "
     $g_i=" "
     $g_j=" "
     }
 if ($p_x -eq 5)
     {
     $g_a=" "
     $g_b=" "
     $g_c=" "
     $g_d=" "
     $g_e=" "
     $g_f=$p_char
     $g_g=" "
     $g_h=" "
     $g_i=" "
     $g_j=" "
     }
 if ($p_x -eq 6)
     {
     $g_a=" "
     $g_b=" "
     $g_c=" "
     $g_d=" "
     $g_e=" "
     $g_f=" "
     $g_g=$p_char
     $g_h=" "
     $g_i=" "
     $g_j=" "
     }
 if ($p_x -eq 7)
     {
     $g_a=" "
     $g_b=" "
     $g_c=" "
     $g_d=" "
     $g_e=" "
     $g_f=" "
     $g_g=" "
     $g_h=$p_char
     $g_i=" "
     $g_j=" "
     }
 if ($p_x -eq 8)
     {
     $g_a=" "
     $g_b=" "
     $g_c=" "
     $g_d=" "
     $g_e=" "
     $g_f=" "
     $g_g=" "
     $g_h=" "
     $g_i=$p_char
     $g_j=" "
     }
 if ($p_x -eq 9)
     {
     $g_a=" "
     $g_b=" "
     $g_c=" "
     $g_d=" "
     $g_e=" "
     $g_f=" "
     $g_g=" "
     $g_h=" "
     $g_i=" "
     $g_j=$p_char
     }
 if (($c_a -eq $true) -and ($g_a -eq $p_char))
     {
         $collision=$true
     }
 if (($c_b -eq $true) -and ($g_b -eq $p_char))
     {
         $collision=$true
     }
 if (($c_c -eq $true) -and ($g_c -eq $p_char))
     {
         $collision=$true
     }
 if (($c_d -eq $true) -and ($g_d -eq $p_char))
     {
         $collision=$true
     }
 if (($c_e -eq $true) -and ($g_e -eq $p_char))
     {
         $collision=$true
     }
 if (($c_f -eq $true) -and ($g_f -eq $p_char))
     {
         $collision=$true
     }
 if (($c_g -eq $true) -and ($g_g -eq $p_char))
     {
         $collision=$true
     }
 if (($c_h -eq $true) -and ($g_h -eq $p_char))
     {
         $collision=$true
     }
 if (($c_i -eq $true) -and ($g_i -eq $p_char))
     {
         $collision=$true
     }
 if (($c_j -eq $true) -and ($g_j -eq $p_char))
     {
         $collision=$true
     }
 if ($collision -eq $true)
     {
         if ($god_mode -eq $false)
         {
         $grid = "||||Collision! Game Over:C||||"
                #"||||||||||||||||||||||||||||||"
         write-host $grid -ForegroundColor Cyan -BackgroundColor Black
         exit
         }
     }
     
 
 if (($o_a -eq $o_char) -and ($g_a -eq " "))
     {
         $g_a=$e_char
     }
 if (($o_b -eq $o_char) -and ($g_b -eq " "))
     {
         $g_b=$e_char
     }
 if (($o_c -eq $o_char) -and ($g_c -eq " "))
     {
         $g_c=$e_char
     }
 if (($o_d -eq $o_char) -and ($g_d -eq " "))
     {
         $g_d=$e_char
     }
 if (($o_e -eq $o_char) -and ($g_e -eq " "))
     {
         $g_e=$e_char
     }
 if (($o_f -eq $o_char) -and ($g_f -eq " "))
     {
         $g_f=$e_char
     }
 if (($o_g -eq $o_char) -and ($g_g -eq " "))
     {
         $g_g=$e_char
     }
 if (($o_h -eq $o_char) -and ($g_h -eq " "))
     {
         $g_h=$e_char
     }
 if (($o_i -eq $o_char) -and ($g_i -eq " "))
     {
         $g_i=$e_char
     }
 if (($o_j -eq $o_char) -and ($g_j -eq " "))
     {
         $g_j=$e_char
     }
 
 if (($c_a -eq $true) -and ($g_a -eq " "))
     {
         $g_a=$o_char
     }
 if (($c_b -eq $true) -and ($g_b -eq " "))
     {
         $g_b=$o_char
     }
 if (($c_c -eq $true) -and ($g_c -eq " "))
     {
         $g_c=$o_char
     }
 if (($c_d -eq $true) -and ($g_d -eq " "))
     {
         $g_d=$o_char
     }
 if (($c_e -eq $true) -and ($g_e -eq " "))
     {
         $g_e=$o_char
     }
 if (($c_f -eq $true) -and ($g_f -eq " "))
     {
         $g_f=$o_char
     }
 if (($c_g -eq $true) -and ($g_g -eq " "))
     {
         $g_g=$o_char
     }
 if (($c_h -eq $true) -and ($g_h -eq " "))
     {
         $g_h=$o_char
     }
 if (($c_i -eq $true) -and ($g_i -eq " "))
     {
         $g_i=$o_char
     }
 if (($c_j -eq $true) -and ($g_j -eq " "))
     {
         $g_j=$o_char
     }
 
 
 $op_pos_3=$op_pos_1
 $op_pos_1=Get-Random -minimum 1 -maximum $range_factor
 if (($op_pos_1 -eq 24) -and ($walls_cfg -eq $true))
     {
         $o_a=$o_char
         $o_b=$o_char
         $o_c=$o_char
         $o_d=$o_char
     }
     else
     {
         $o_a=" "
         $o_b=" "
         $o_c=" "
         $o_d=" "
     }
 
 if (($op_pos_1 -eq 22) -and ($walls_cfg -eq $true))
     {
         $o_g=$o_char
         $o_h=$o_char
         $o_i=$o_char
         $o_j=$o_char
     }
     else
     {
         $o_g=" "
         $o_h=" "
         $o_i=" "
         $o_j=" "
     }
 if ($op_pos_1 -eq 2)
     {
         $o_a=$o_char
     }
     else
     {
         $o_a=" "
     }
 if ($op_pos_1 -eq 4)
     {
         $o_b=$o_char
     }
     else
     {
         $o_b=" "
     }
 if ($op_pos_1 -eq 2)
     {
         $o_c=$o_char
     }
     else
     {
         $o_c=" "
     }
 if ($op_pos_1 -eq 6)
     {
         $o_d=$o_char
     }
     else
     {
         $o_d=" "
     }
 if ($op_pos_1 -eq 8)
     {
         $o_e=$o_char
     }
     else
     {
         $o_e=" "
     }
 if ($op_pos_1 -eq 10)
     {
         $o_f=$o_char
     }
     else
     {
         $o_f=" "
     }
 if ($op_pos_1 -eq 12)
     {
         $o_g=$o_char
     }
     else
     {
         $o_g=" "
     }
 if ($op_pos_1 -eq 14)
     {
         $o_h=$o_char
     }
     else
     {
         $o_h=" "
     }
 if ($op_pos_1 -eq 16)
     {
         $o_i=$o_char
     }
     else
     {
         $o_i=" "
     }
 if ($op_pos_1 -eq 18)
     {
         $o_j=$o_char
     }
     else
     {
         $o_j=" "
     }
 
 if (($op_pos_3 -eq 24) -and ($walls_cfg -eq $true))
     {
         $c_a=$true
         $c_b=$true
         $c_c=$true
         $c_d=$true
     }
     else
     {
         $c_a=$false
         $c_b=$false
         $c_c=$false
         $c_d=$false
     }
 
 if (($op_pos_3 -eq 22) -and ($walls_cfg -eq $true))
     {
         $c_g=$true
         $c_h=$true
         $c_i=$true
         $c_j=$true
     }
     else
     {
         $c_g=$false
         $c_h=$false
         $c_i=$false
         $c_j=$false
     }
 
 if ($op_pos_3 -eq 2)
     {
         $c_a=$true
     }
     else
     {
         $c_a=$false
     }
 if ($op_pos_3 -eq 4)
     {
         $c_b=$true
     }
     else
     {
         $c_b=$false
     }
 if ($op_pos_3 -eq 2)
     {
         $c_c=$true
     }
     else
     {
         $c_c=$false
     }
 if ($op_pos_3 -eq 6)
     {
         $c_d=$true
     }
     else
     {
         $c_d=$false
     }
 if ($op_pos_3 -eq 8)
     {
         $c_e=$true
     }
     else
     {
         $c_e=$false
     }
 if ($op_pos_3 -eq 10)
     {
         $c_f=$true
     }
     else
     {
         $c_f=$false
     }
 if ($op_pos_3 -eq 12)
     {
         $c_g=$true
     }
     else
     {
         $c_g=$false
     }
 if ($op_pos_3 -eq 14)
     {
         $c_h=$true
     }
     else
     {
         $c_h=$false
     }
 if ($op_pos_3 -eq 16)
     {
         $c_i=$true
     }
     else
     {
         $c_i=$false
     }
 if ($op_pos_3 -eq 18)
     {
         $c_j=$true
     }
     else
     {
         $c_j=$false
     }
 if ($time -ge 150)
     {
         $walls_cfg = $true
     }
 if ($time -eq 100)
     {
         $range_factor = 45 
     }
 if ($time -eq 200)
     {
         $range_factor = 40
     }
 if ($time -eq 300)
     {
         $range_factor = 30
     }
 $time++
 $delay_total=$delay_total-(($time/2)/4)
 if ($delay_total -le 50)
     {
     $delay_total=50
     }
 $delay_total=[math]::Round($delay_total)
 $g_time="|{0:00000}" -f $time
 write-host ">" -NoNewline -ForegroundColor Black -BackgroundColor Gray
 $grid_op="{0}{1}{2}{3}{4}{5}{6}{7}{8}{9}" -f $o_a, $o_b, $o_c, $o_d, $o_e, $o_f, $o_g, $o_h, $o_i, $o_j
 write-host $grid_op -NoNewline -ForegroundColor DarkRed -BackgroundColor White
 write-host "|" -NoNewline -ForegroundColor DarkRed -BackgroundColor White
 write-host " " -NoNewline -ForegroundColor Black -BackgroundColor Gray
 write-host "|" -NoNewline -ForegroundColor Black -BackgroundColor White
 $grid="{0}{1}{2}{3}{4}{5}{6}{7}{8}{9}" -f $g_a, $g_b, $g_c, $g_d, $g_e, $g_f, $g_g, $g_h, $g_i, $g_j
 write-host $grid -NoNewline -ForegroundColor Cyan -BackgroundColor Black
 write-host $g_time -ForegroundColor Black -BackgroundColor White 
 Start-Sleep -Milliseconds $delay_total
     }
 }
`

