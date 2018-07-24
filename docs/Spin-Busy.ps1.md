---
Author: joe otto
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 157
Published Date: 2008-03-09t21
Archived Date: 2017-04-30t12
---

# spin-busy - 

## Description

spin-busy displays a “spinning” character with each step reflecting an individual pipeline object being passed through.

## Comments



## Usage



## TODO



## function

`spin-busy`

## Code

`#
 #
 function Spin-Busy {
 	param(
 		[Int]														$msWait =		0,
 		[String]													$spinStr =		'-\|/. ',
 		[Char[]]													$spinChars =	[Char[]] ($spinStr.ToCharArray()),
 		[System.Management.Automation.Host.PSHostRawUserInterface]	$rawUI =		(Get-Host).UI.RawUI,
 		[ConsoleColor]												$bgColor =		$rawUI.BackgroundColor,
 		[ConsoleColor]												$fgColor =		$rawUI.ForegroundColor,
 		[System.Management.Automation.Host.Coordinates]				$curPos =		$rawUI.Get_CursorPosition(),
 		[Int32]														$startX =		$curPos.X,
 		[Int32]														$maxX =			$rawUI.WindowSize.Width,
 		[Switch]													$trails
 	)
 
 	begin {
 		$trailCell =	$rawUI.NewBufferCellArray(@($spinChars[-2]), $fgColor, $bgColor);
 		$blankCell =	$rawUI.NewBufferCellArray(@($spinChars[-1]), $fgColor, $bgColor);
 
 		$spinCells =	$spinChars[0..($spinChars.Count - 3)];
 
 		for ($i=0; $i -lt ($spinCells.Count); ++$i) {
 			$spinCells[$i] = $rawUI.NewBufferCellArray(@($spinCells[$i]), $fgColor, $bgColor)
 		}
 
 		$charNdx =	0;
 	}
 
 	process {
 		if ($charNdx -lt $spinCells.Count)	{					$rawUI.SetBufferContents($curPos, $spinCells[$charNdx++]);					}
 		else								{ $charNdx = 0;		$rawUI.SetBufferContents($curPos, $trailCell);
 			if ($trails) {
 				if (++$curPos.X -gt $maxX) 	{ do { --$curPos.X;	$rawUI.SetBufferContents($curPos, $blankCell)	} until ($curPos.X -le $startX)	}
 			}
 		}
 
 		Start-Sleep -milliseconds $msWait
 		$_
 	}
 
 	end {
 		do { $rawUI.SetBufferContents($curPos, $blankCell);	}
 		until (--$curPos.X -le $startX)
 	}
 }
`

