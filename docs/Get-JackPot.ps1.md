---
Author: craig dayton
Publisher: 
Copyright: 
Email: 
Version: 2.1.1
Encoding: ascii
License: cc0
PoshCode ID: 6828
Published Date: 2017-04-01t20
Archived Date: 2017-04-07t18
---

# get-jackpot - 

## Description

powershell script for displaying and picking washington state lottery game winning numbers.

## Comments



## Usage



## TODO



## script

`convert-filetocsv`

## Code

`#
 #
 
   .VERSION 2.1.1
   .GUID 0fd916fe-3a0d-48c4-a196-18ea085e071f
   .AUTHOR Craig Dayton
   .COMPANYNAME Example.com
   .COPYRIGHT Absolute Zero
   .TAGS 
   .LICENSEURI 
   .PROJECTURI https://github.com/cadayton/JackPot
   .ICONURI 
   .EXTERNALMODULEDEPENDENCIES 
   .REQUIREDSCRIPTS 
   .EXTERNALSCRIPTDEPENDENCIES 
   .RELEASENOTES
 
 #>
 
 <#
   .SYNOPSIS
     Display the current Washington State lottery game results and optionally stores the 
     results in a local history file.
 
   .DESCRIPTION
     Displays Washington lottery game results by invoking a web request to
     http://www.walottery.com/WinningNumbers and extracts the game results
     onto the console and optionally saves the results locally.
 
     Picks game winning numbers based on frequency of past winning numbers.
 
   .PARAMETER game
     Displays all the history results of a specified lottery game in the Out-Gridview.
 
     When used in combination with the '-picker' option, will select a set numbers to
     be played for the specified game.
 
   .PARAMETER online
     Displays the current lottery game results on the console.
 
   .PARAMETER update
     Displays the current lottery game results on the console and updates
     the history file.
 
     When used in combination with '-online' option, will compare the winning
     numbers to the numbers selected using the '-picker' and report on the number 
     of matches.
 
   .PARAMETER all
     Displays all the history lottery game results in a Out-Gridview.
 
   .PARAMETER picker
     Used in combination with the 'game' option to pick a set of winning
     numbers for a game based on frequency of past winning numbers.
 
     If the '-all' option is used in combintion with the '-picker' option then
     then all potential game numbers will be candidates for selection.
 
     To restore the number selection process back to only selecting from
     the most frequently used numbers for a game, the file, JackPot-HotNums.csv
     must be deleted.
 
   .PARAMETER picks
     Displays the history of past games that have been played.
 
 
   .PARAMETER count
     Used in combination with the 'game' and 'picker' option to generate
     a specified set of numbers.  The default is 1.
 
   .INPUTS
     A history csv file named, JackPot-Results.csv
 
     http://www.walottery.com/WinningNumbers
 
   .OUTPUTS
     A history csv file named, JackPot-Results.csv
 
   .EXAMPLE
 
     The ordering of the examples below is the typically workflow
     when placing lottery game bets.
     
     It is not necessary to actually purchase any game tickets, but one can
     just play along as though a purchase was made.
 
     I doubt this code is providing any increase in the odds of winning any 
     specific game.
     
     It will though greatly reduce the time it takes to sort out any winning
     matches from multiple tickets.  Provided one is using this code to 
     select the game numbers.
 
   .EXAMPLE
     Get-JackPot -update
 
     Queries the lottery web page and then extracts and displays the
     current game results.  The history file, JackPot-Result.csv is then
     updated with new game results.
 
     Routinely running this command build a local copy of all the lottery
     games.
 
   .EXAMPLE
     Get-JackPot -update -picker -game PowerBall -count 2
 
       Generates 2 sets winning numbers for the 'PowerBall' game
       and places the bets into the file, JackPot-Picks.csv.
       
       The numbers are randomly selected from the game entries in the
       file, JackPot-HotNums.csv.  This file consists of the most
       frequently selected winning numbers for the game.
 
       These numbers need to be generated on the same day as the
       drawing is being held.
 
     Get-JackPot -update -picker -all -game MegaMillions
 
       The '-all' option sets the default selection of game numbers
       to be all possible numbers for the game.
 
       Delete the file, JackPot-HotNums.csv to restore the game selection
       of numbers to the most frequently selected numbers for the game.
 
     Example Output:
       PowerBall Game (1):  16 23 25 32 64 09
       PowerBall Game (2):  25 28 40 52 64 21
 
     With these numbers in hand, go to your local lottery store
     and complete the game card.
 
     By default only the most frequent winning numbers for the game
     are candidiates for selection.  The default selection process
     can be overridden by specified '-all' option.
 
     Once the '-all' is specified that method becomes the new default.
     To switch back to the prior default the file, JackPot-HotNums.csv
     must be deleted.
 
     Valid games are 'PowerBall', 'MegaMillions', 'Lotto', 'Hit5', 'Match4',
     and 'DailyGame'.
 
   .EXAMPLE
     Get-JackPot -online -update
 
     Queries the lottery web page and then extracts and displays the
     current game results.
     
     Next the winning results are compared to the picked numbers for the
     game and a report is generated showing of balls matched per game.
 
     The recent game picks in the file, JackPot-Picks.csv are updated
     with the winning results and match count then moved to the file,
     JackPot-PickHostory.csv. Afterwards, the JackPot-Picks.csv file is
     removed.
   
   .EXAMPLE
     Get-JackPot
 
     Displays the last 12 game history records on the console.
 
   .EXAMPLE
     Get-JackPot -game PowerBall
 
     Displays all the PowerBall records in the history file. 
 
   .EXAMPLE
     Get-JackPot -online
 
     Queries the lottery web page and then extracts and displays the
     current game results.
 
   .EXAMPLE
     Get-JackPot -picks
 
     Displays history of the games played compared to the drawing results.
 
   .NOTES
     Author: Craig Dayton
       2.1.1: 04/01/2017 - Fixed logic errors & updated embeded documentation
       2.1.0: 03/27/2017 - Added feature to evaluate picked numbers against winning numbers
       2.0.0: 03/24/2017 - Added feature to generate a set of winning numbers
       1.0.2: 03/24/2017 - Game record duplication algorthim modified
       1.0.1: 03/23/2017 - Fixed some logic errors
       1.0.0: 03/22/2017 - initial release.
     
 #>
 
 
   [cmdletbinding()]
     Param(
       [Parameter(Position=0,
         Mandatory=$false,
         HelpMessage = "Enter a lottery game name (i.e. PowerBall)",
         ValueFromPipeline=$True)]
         #[ValidateNotNullorEmpty("^[a-zA-Z]{12}$")]
         [string]$game,
       [Parameter(Position=1,
         Mandatory=$false,
         HelpMessage = "Display Online Lottery Results",
         ValueFromPipeline=$True)]
         [ValidateNotNullorEmpty()]
         [switch]$online,
       [Parameter(Position=2,
         Mandatory=$false,
         HelpMessage = "Display Online Lottery Results & update history file",
         ValueFromPipeline=$True)]
         [ValidateNotNullorEmpty()]
         [switch]$update,
       [Parameter(Position=3,
         Mandatory=$false,
         HelpMessage = "Display all game history file records",
         ValueFromPipeline=$True)]
         [ValidateNotNullorEmpty()]
         [switch]$all,
       [Parameter(Position=4,
         Mandatory=$false,
         HelpMessage = "Display all game history file records",
         ValueFromPipeline=$True)]
         [ValidateNotNullorEmpty()]
         [switch]$picker,
       [Parameter(Position=5,
         Mandatory=$false,
         HelpMessage = "Display the pick history of all games",
         ValueFromPipeline=$True)]
         [ValidateNotNullorEmpty()]
         [switch]$picks,
       [Parameter(Position=6,
         Mandatory=$false,
         HelpMessage = "Number of games",
         ValueFromPipeline=$True)]
         [ValidateNotNullorEmpty()]
         [int]$count = 1
    )
 #
 
   $URI1 = "http://www.walottery.com/WinningNumbers";
 
   [String[]]$JackPotHeader  = "Game", "DrawDate","DrawDay", "Numbers", "Multiplier", "JackPot", "NextDraw", "NextDay";
   [String[]]$MultiHeader    = "Game", "HotNums","Multiplier";
   [String[]]$StdHeader      = "Game", "HotNums";
   [String[]]$DailyHeader    = "Game", "Pos1","Pos2","Pos3";
   [String[]]$PickHeader     = "Game", "PickDate","PickDay","Choices","Cost","WinNums","Matches","Prize","Multiplier";
 
     $HotArray = New-Object System.Collections.ArrayList;
     $HotArray.Add('PowerBall,03 12 16 23 25 28 32 33 40 52 64 69,02 03 05 06 09 10 12 17 19 20 21 25') | Out-Null;
     $HotArray.Add('MegaMillions,02 11 20 25 29 31 35 41 44 45 49 51,01 02 03 04 06 07 08 09 10 12 14 15') | Out-Null;
     $HotArray.Add('Lotto,28 26 03 37 47 13 17 27 39 49 19 25 43 21 20 08 41 12 01 24 10') | Out-Null;
     $HotArray.Add('Hit5,35 37 13 33 14 23 17 12 27 07 28 02 21 03 11 34 38 10 31') | Out-Null;
     $HotArray.Add('Match4,19 18 24 05 13 08 04 02 16 07 21 06') | Out-Null;
     $HotArray.Add('DailyGame,8 5 4 7 1,7 2 9 6 5,8 0 1 2 4') | Out-Null;
   #
 
     $AllArray = New-Object System.Collections.ArrayList;
     $AllArray.Add('PowerBall,01 02 03 04 05 06 07 08 09 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 61 62 63 64 65 66 67 68 69,01 02 03 04 05 06 07 08 09 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26') | Out-Null;
     $AllArray.Add('MegaMillions,01 02 03 04 05 06 07 08 09 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 61 62 63 64 65 66 67 68 69 70 71 72 73 74 75,01 02 03 04 06 07 08 09 10 12 14 15') | Out-Null;
     $AllArray.Add('Lotto,01 02 03 04 05 06 07 08 09 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49') | Out-Null;
     $AllArray.Add('Hit5,01 02 03 04 05 06 07 08 09 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39') | Out-Null;
     $AllArray.Add('Match4,01 02 03 04 05 06 07 08 09 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24') | Out-Null;
     $AllArray.Add('DailyGame,1 2 3 4 5 6 7 8 9,1 2 3 4 5 6 7 8 9,1 2 3 4 5 6 7 8 9') | Out-Null;
   #
 
 #
 
 
   function Convert-fileToCSV {
     param ([string]$fname)
 
     $bufLine  = $null;
     $game = $Global:game;
     [bool]$needNums = $true;
     $DrawDate, $DrawDay, $Numbers, $Multiplier, $Prize, $NextDate, $NextDay = $null;
 
       if ($_ -notmatch "^ ") {
         $curLine = $_;
         
         if ($curLine -match "Latest Draw:" -and $DrawDate -eq $null) {
           $junk,$ld = $curline.Split(":");
           $day,$date = $ld.Split("/");
           $DrawDay = $day;
           $DrawDate = Get-Date $date -format yyyy-MM-dd;
         }
 
         if ($curLine -match "^\d" -and $needNums) {
           $Numbers += $curLine + " ";
         }
 
         if ($curLine -match "Power Play" -and $Multiplier -eq $null) {
           $Multiplier = $curLine.Substring(11,3);
         }
 
         if ($curLine -match "Megaplier" -and $Multiplier -eq $null) {
           $Multiplier = $curLine.Substring(10,3);
         }
 
         if ($curLine -match "[$]" -and $Prize -eq $null) {
           $needNums = $false;
           $Prize = $curLine;
         }
 
         if ($curLine -match "Next Draw:" -and $NextDate -eq $null) {
           $junk,$ld = $curline.Split(":");
           $day,$date = $ld.Split("/");
           if ($date -ne $null ) {
             $NextDay = $day;
             $NextDate = Get-Date $date -format yyyy-MM-dd;
           } else {
             $NextDate = $day;
           };
         }
 
         if ($curLine -match "Daily" -and $NextDate -eq $null) { $NextDate = $curLine};
 
       }
     }
 
     $bufLine  = $game + ";" + $DrawDate + ";" + $DrawDay + ";" + $Numbers + ";";
     $bufLine += $Multiplier + ";" + $Prize + ";" + $NextDate + ";" + $NextDay;
     $bufLine | Out-File -FilePath $temp2 -Append -Encoding ascii;
 
     if (Test-Path $fname) { Remove-Item -Path $fname };
 
   }
 
   function Update-JackPotHistory {
     param ([string]$fname)
 
     if (Test-Path $JackPot ) {
     
       $histLine  = Get-Content -Path $JackPot | Select-Object -Last 1;
       $currLine  = Get-Content -Path $fname | Select-Object -Last 1;
 
         $histLine  = Get-Content -Path $JackPot | Select-Object -Last 12;
           $curLine    = $_;
           $curLine24  = $_.Substring(0,23);
           [bool]$duplicate = $false;
           $histLine | ForEach-Object {
             $histLine24 = $_.Substring(0,23);
             if ($curLine24 -eq $histLine24) {
               $duplicate = $true;
             }
           }
           if ($duplicate) {} else {
             $curLine | Out-File -FilePath $JackPot -Append -Encoding ascii;
           }
         }
       }
     } else {
         $_ | Out-File -FilePath $JackPot -Append -Encoding ascii;
       }
     }
   }
 
   function Get-Matches {
     param([string]$pickGame, [string]$choices, [string]$winners)
 
     $c1 = $choices.Split(" ");
     $w1 = $winners.Split(" ");
     [int]$matchcnt    = 0;
     [int]$gameBall    = 0;
     [int]$lastIdx     = $w1.Length - 1;
     [int[]]$matchRslt = 0,0;
 
     for ($i = 0;$i -le $lastIdx; $i++) {
       for ($j = 0; $j -le $lastIdx; $j++) {
         if ($c1[$i] -eq $w1[$j]) {
           $matchcnt++
           if ($pickGame -eq "PowerBall" -or $pickGame -eq "MegaMillions") {
             if ($i -eq $lastIdx -and $j -eq $lastIdx) {$gameBall++; $matchcnt--; }
           }
         }
       }
     }
 
     $matchRslt[0] = $matchcnt;  $matchRslt[1] = $gameBall;
     return $matchRslt;
 
   }
 
   function Update-PickHistory {
     param ([string]$fname)
 
     if (Test-Path $curPicks ) {
     
       $currentGames = Import-CSV -Path $fname -Delimiter ";" -Header $JackPotHeader;
       $currentPicks = Import-CSV -Path $curPicks -Delimiter ";" -Header $PickHeader;
       $pickResults = @();
 
       Write-Host "Checking prior games for matching numbers..." -ForegroundColor Yellow;
 
       $currentGames | ForEach-Object {
         [bool]$gameBallMatch = $false;
         $curGame = $_;
         $curGameName = $_.Game + $_.DrawDate;
         #[int]$cmpSize = ($gameName.Length + 11) - 1;
         #$curGameDate = $_DrawDate;
         $currentPicks | ForEach-Object {
           $curPick = $_;
           $curPickName = $_.Game + $_.PickDate;
           if ($curGameName -eq $curPickName) {
             $curPick.WinNums = $curGame.Numbers;
               $nums = $curGame.Numbers;
               $cnt = $nums.Split(" ")
               $curPick.Matches = $cnt.Length;
               $gameBallMatch = $true;
               $color = "Red";
               $matches = Get-Matches $curPick.Game $curPick.Choices $curPick.WinNums;
               $curPick.Matches = $matches[0];
               if ($matches[0] -gt 0) { $color = "Green"} else { $color = "Blue"}
               if ($matches[1] -gt 0) { $gameBallMatch = $true; $color1 = "Green"} else {$color1 = "Blue"}
             }
             if ($curPick.Game -eq "PowerBall" -or $curPick.Game -eq "MegaMillions" ) {
               $curPick.Multiplier = $curGame.Multiplier;
             } else { 
               $curPick.Multiplier = "NA";
             };
             $curPick | Export-CSV -Path $PickHis -Delimiter "," -Append -NoTypeInformation
             $pickResults += $curPick;
           }
         }
       }
       $pickResults | Format-Table -Auto
       Write-Host "Check $URI1 for the amount won for each match" -ForegroundColor Green
       Remove-Item -Path $curPicks
     }
   }
 
   function Get-WaWebRequest {
     
     $response = Invoke-WebRequest -URI $URI1;
 
     if ($response.StatusCode -eq "200") {
       Write-Progress -Activity "Processing response from $URI1 ..." -Status "Please wait"
 
       $Global:game = "PowerBall";
       Write-Host " Processing $Global:game results"  -ForegroundColor Green
       $data = $($response.ParsedHtml.getElementsByTagName("div") |
                 Where-Object classname -eq "game-bucket game-bucket-powerball"
               );
       $data.innerText | Out-File -FilePath $temp1 -Append -Encoding ascii;
       Convert-fileToCSV $temp1;
 
       $Global:game = "MegaMillions";
       Write-Host " Processing $Global:game results"  -ForegroundColor Green
       $data = $($response.ParsedHtml.getElementsByTagName("div") |
                 Where-Object classname -eq "game-bucket game-bucket-megamillions"
               );
       $data.innerText | Out-File -FilePath $temp1 -Append -Encoding ascii;
       Convert-fileToCSV $temp1;
 
       Write-Progress -Activity "Done" -Completed;
 
       $Global:game = "Lotto";
       Write-Host " Processing $Global:game results"  -ForegroundColor Green
       $data = $($response.ParsedHtml.getElementsByTagName("div") |
                 Where-Object classname -eq "game-bucket game-bucket-lotto"
               );
       $data.innerText | Out-File -FilePath $temp1 -Append -Encoding ascii;
       Convert-fileToCSV $temp1;
 
       $Global:game = "Hit5";
       Write-Host " Processing $Global:game results"  -ForegroundColor Green
       $data = $($response.ParsedHtml.getElementsByTagName("div") |
                 Where-Object classname -eq "game-bucket game-bucket-hit5"
               );
       $data.innerText | Out-File -FilePath $temp1 -Append -Encoding ascii;
       Convert-fileToCSV $temp1;
 
       $Global:game = "Match4";
       Write-Host " Processing $Global:game results"  -ForegroundColor Green
       $data = $($response.ParsedHtml.getElementsByTagName("div") |
                 Where-Object classname -eq "game-bucket game-bucket-match4"
               );
       $data.innerText | Out-File -FilePath $temp1 -Append -Encoding ascii;
       Convert-fileToCSV $temp1;
 
       $Global:game = "DailyGame";
       Write-Host " Processing $Global:game results"  -ForegroundColor Green
       $data = $($response.ParsedHtml.getElementsByTagName("div") |
                 Where-Object classname -eq "game-bucket game-bucket-dailygame"
               );
       $data.innerText | Out-File -FilePath $temp1 -Append -Encoding ascii;
       Convert-fileToCSV $temp1;
 
         $Global:game = "Keno"; $Global:ID = 7;
         Write-Host " Processing $Global:game results"  -ForegroundColor Green
         $data = $($response.ParsedHtml.getElementsByTagName("div") |
                   Where-Object classname -eq "game-bucket game-bucket-keno"
                 );
         $data.innerText | Out-File -FilePath $temp1 -Append -Encoding ascii;
         Convert-fileToCSV $temp1;
       #>
 
       $currentGames = Import-CSV -Path $temp2 -Delimiter ";" -Header $JackPotHeader;
       $currentGames | Format-Table -AutoSize -Wrap
 
       if ($update) { Update-JackPotHistory $temp2; };
       if ((Test-Path $curPicks) -and $online -and $update) { Update-PickHistory $temp2 };
 
       if (Test-Path $temp2 ) {
         Remove-Item -Path $temp2 
       };
 
     } else {
       Write-Host "Received error code: $response.StatusCode from $URI1";
     }
 
   }
 
   function Show-JackPotError {
     Write-Host "$JackPot not found"
     Write-Host "Execute 'Get-JackPot -update' to create the history file."
     Write-Host "Get-Help Get-JackPot -full and review the documentation."
   }
 
   function Get-MultiPicker {
     $HotPB = Import-CSV -Path $HotNums -Delimiter "," -Header $MultiHeader |
       Where-Object {$_.Game -match $game };
     for ($i = 1; $i -le $count; $i++) {
       $topNums  = $HotPB.HotNums;
       $topMulti = $HotPB.Multiplier;
       $top1     = $topNums.Split(" ");
       $top2     = $topMulti.Split(" ");
       $sel1     = Get-Random -InputObject $top1 -Count 5
       $sel2     = Get-Random -InputObject $top2 -Count 1
       $sela     = $sel1 | Sort-Object;
       $selb     = [system.String]::Join(" ",$sela)
       $sel      = $selb + " " + $sel2;
       Write-Host "$game Game ($i):  $sel" -ForegroundColor Green
       if ($update) {
         $dt = Get-Date -format "yyyy-MM-dd";
         $da = Get-Date -uformat %a;
         $da = " " + $da.ToUpper();
         $grec = $game + ";" + $dt + ";" + $da + ";" + $sel + ";;;";
         $grec | Out-File -FilePath $curPicks -Append -Encoding ascii;
       }
     }
   }
 
   function Get-NumPicker {
     Param ([int]$numcnt)
 
     $HotPB = Import-CSV -Path $HotNums -Delimiter "," -Header $StdHeader |
       Where-Object {$_.Game -match $game };
 
     for ($i = 1; $i -le $count; $i++) {
       $topNums  = $HotPB.HotNums;
       $top1     = $topNums.Split(" ");
       $sel1     = Get-Random -InputObject $top1 -Count $numcnt
       $sela     = $sel1 | Sort-Object;
       $selb     = [system.String]::Join(" ",$sela)
       Write-Host "$game Game ($i):  $selb" -ForegroundColor Green
       if ($update) {
         $dt = Get-Date -format "yyyy-MM-dd";
         $da = Get-Date -uformat %a;
         $da = " " + $da.ToUpper();
         $grec = $game + ";" + $dt + ";" + $da + ";" + $selb + ";;;";
         $grec | Out-File -FilePath $curPicks -Append -Encoding ascii;
       }
     }
   }
 
   function Get-GameNumbers {
 
     if ($all) {
       if (Test-Path $HotNums) { Remove-Item -Path $HotNums }
       $AllArray | ForEach-Object {
         $_ | Out-File -FilePath $HotNums -Append -Encoding ascii;
       }
     }
 
     if (Test-Path $HotNums) {} else { 
       $HotArray | ForEach-Object {
         $_ | Out-File -FilePath $HotNums -Append -Encoding ascii;
       }
     }
 
     switch ($game) {
       "PowerBall" {
         Get-MultiPicker;
       }
       "MegaMillions" {
         Get-MultiPicker;
       }
       "Lotto" {
         Get-NumPicker 6
       }
       "Hit5" {
         Get-NumPicker 5
       }
       "Match4" {
         Get-NumPicker 4
       }
       "DailyGame" {
         $HotPB = Import-CSV -Path $HotNums -Delimiter "," -Header $DailyHeader |
           Where-Object {$_.Game -match $game };
         for ($i = 1; $i -le $count; $i++) {
           $topPos1  = $HotPB.Pos1;
           $topPos2  = $HotPB.Pos2;
           $topPos3  = $HotPB.Pos3;
           $pos1     = $topPos1.Split(" ");
           $pos2     = $topPos2.Split(" ");
           $pos3     = $topPos3.Split(" ");
           $sel1     = Get-Random -InputObject $pos1 -Count 1
           $sel2     = Get-Random -InputObject $pos2 -Count 1
           $sel3     = Get-Random -InputObject $pos3 -Count 1
           $sel      = $sel1 + " " + $sel2 + " " + $sel3;
           Write-Host "$game Game ($i):  $sel" -ForegroundColor Green
           if ($update) {
             $dt = Get-Date -format "yyyy-MM-dd";
             $da = Get-Date -uformat %a;
             $da = " " + $da.ToUpper();
             $grec = $game + ";" +$dt + ";" + $da + ";" + $sel + ";;;";
             $grec | Out-File -FilePath $curPicks -Append -Encoding ascii;
           }
         }
 
       }
       Default {
         Write-Host "$game not found" -ForegroundColor Red
         Write-Host "Valid game names are: 'PowerBall, MegaMillions, Hit5, Match4, and DailyGame' " -ForegroundColor Green
       }
     }
 
   }
 
 #
 
 
   $sPath    = Get-Location;
   $temp1    = "$sPath\temp1.txt";
   $temp2    = "$sPath\temp2.txt";
   $JackPot  = "$sPath\JackPot-Results.csv";
   $HotNums  = "$sPath\JackPot-HotNums.csv";
   $curPicks    = "$sPath\JackPot-Picks.csv";
   $PickHis  = "$sPath\JackPot-PickHistory.csv";
 
   if (Test-Path $temp1 ) {
     Remove-Item -Path $temp1 
   };
 
   if (Test-Path $temp2 ) {
     Remove-Item -Path $temp2 
   };
 
   $choice = $null;
   if ($game -and $picker) { $choice = "GamePicker"} 
   elseif ($online -or $update) { $choice = "WebRequest"}
   elseif ($game) { $choice = "GameHistory"}
   elseif ($all) { $choice = "AllHistory"}
   elseif ($picks) { $choice = "PickHistory"}
 
   switch ($choice) {
     "WebRequest" {
       Get-WaWebRequest;
     }
     "GameHistory" {
       if (Test-Path $JackPot) {
         $currentGames = Import-CSV -Path $JackPot -Delimiter ";" -Header $JackPotHeader |
           Where-Object {$_.Game -match $game };
         if ($currentGames) { 
           $currentGames |  Out-GridView -Title "$game at $URI1"
         } else {
           Write-Host "$game not found" -ForegroundColor Red
           Write-Host "Valid game names are: 'PowerBall, MegaMillions, Hit5, Match4, and DailyGame' " -ForegroundColor Green
        } 
       } else { Show-JackPotError; }
 
     }
     "GamePicker" {
       Get-GameNumbers;
     }
     "AllHistory" {
       if (Test-Path $JackPot) {
         Import-CSV -Path $JackPot -Delimiter ";" -Header $JackPotHeader | Out-GridView -Title "Listing of lottery game records"
       } else { Show-JackPotError; }
     }
     "PickHistory" {
       if (Test-Path $JackPot) {
         $currentPicks = Import-CSV -Path $PickHis
         #$currentPicks | Select-Object | Format-Table -AutoSize -Wrap;
         $currentPicks | Out-GridView -Title "Listing of games played history"
       } else { Show-JackPotError; }
     }
     Default {
       if (Test-Path $JackPot) {
         $currentGames = Import-CSV -Path $JackPot -Delimiter ";" -Header $JackPotHeader;
         $currentGames | Select-Object -Last 12 | Format-Table -AutoSize -Wrap;
       } else { Show-JackPotError; }
     }
   }
 
 #
`

