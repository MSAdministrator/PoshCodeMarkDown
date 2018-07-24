---
Author: zefram
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5845
Published Date: 2015-05-05t14
Archived Date: 2015-05-08t10
---

# mandelbrot set - 

## Description

mandelbrot poc

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $i=[float]-16;
 $j=[float]-16;
 $r=[float]-16;
 $x=[float]-16;
 $y=[float]-16;
 
 $colors="Black","DarkBlue","DarkGreen","DarkCyan","DarkRed","DarkMagenta","DarkYellow","Gray","DarkGray","Blue","Green","Cyan","Red","Magenta","Yellow","White" 
 
 while(($y++) -lt 15) 
 { 
     for($x=0; ($x++) -lt 70; Write-Host " " -BackgroundColor ($colors[$k -band 15]) -NoNewline) 
 		{ 
 			$i=[float]0;
 			$k=[float]0;
 			$r=[float]0; 
 	        do
 			{
 				$j=$r*$r-$i*$i-2+$x/25; $i=2*$r*$i+$y/10; $r=$j
 			}
 	        while (($j*$j+$i*$i) -lt 4 -band ($k++) -lt 111) 
 	    } 
     Write-Host " "
 }
`

