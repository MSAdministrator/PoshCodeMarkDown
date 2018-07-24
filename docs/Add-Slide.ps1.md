---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.6
Encoding: ascii
License: cc0
PoshCode ID: 1899
Published Date: 
Archived Date: 2011-01-14t22
---

# add-slide - 

## Description

snipped of my powerpoint movie creator. this is not general purpose code only a sample for entertainment. scrubbed to take out some meaningful strings, i’ll try to post something better another time.

## Comments



## Usage



## TODO



## function

`add-slide`

## Code

`#
 #
 function Add-Slide($pres, $maxShared, $vmUsages, $vmShared, $maxVM, $vmLoad) {
 	$slide = $pres.slides.range(1).duplicate()
 	#$totalSlides = ($pres.slides | Measure-Object).Count
 	#$pres.slides.range(2).moveto($totalSlides)
 
 	$i = 1
 	foreach ($position in $xOffsets[1..5]) {
 		$shape = $slide.shapes.addShape(5, $position*72, $vmBoxYOffset*72, $vmBoxSize[0]*72, $vmBoxSize[1]*72)
 
 		$shape.textframe2.textrange.text = "XXX"
 		$shape.textframe2.textrange.font.size = 24
 		$shape.line.visible = 0
 
 		$load = $vmLoad[$i-1]
 		$H = 81 - (81 * $load / 100)
 		$S = 63 + (46 * $load / 100)
 		$shape.fill.forecolor.rgb = From-Hsl $H $S 138
 
 		$shape.fill.OneColorGradient(1, 2, 0.6)
 		$i++
 	}
 
 	$shared = ($vmShared | measure-object -sum).Sum
 	$perSize = $shared / $maxShared
 	$maxHeight = ($memBoxYBottom - $memBoxYTop)
 	$height = $maxHeight * $perSize
 	$yOffset = $memBoxYTop + ($maxHeight * (1 - $perSize))
 	$shape = $slide.shapes.addShape(1, $xOffsets[0]*72, $yOffset*72, $vmBoxSize[0]*72, $height*72)
 	$shape.line.visible = 0
 
 	if ($perSize -gt 0.25) {
 		$shape.textframe2.textrange.text = "Xyzzy"
 		$shape.textframe2.textrange.font.size = 24
 	}
 
 	$shape.fill.forecolor.rgb = 5880731
 
 	$shape.fill.OneColorGradient(1, 2, 0.6)
 
 	foreach ($i in 0..4) {
 		$profile = @($vmShared[$i], $vmUsages[$i])
 		$vmTotal = $profile[0] + $profile[1]
 		$vmPercent = $vmTotal / $maxVM
 
 		$maxHeight = ($memBoxYBottom - $memBoxYTop)
 		$height = $maxHeight * $vmPercent
 		$height1 = $height * ($profile[0] / $vmTotal)
 		$height2 = $height * ($profile[1] / $vmTotal)
 		$yOffset1 = $memBoxYTop + ($maxHeight * (1 - $vmPercent))
 		$yOffset2 = $yOffset1 + $height1
 
 		$shape = $slide.shapes.addShape(1, $xOffsets[$i+1]*72, $yOffset1*72, $vmBoxSize[0]*72, $height1*72)
 		$shape.line.visible = 0
 		if ($height1 -gt 0.4) {
 			$shape.textframe2.textrange.text = "Something"
 			$shape.textframe2.textrange.font.size = 24
 		}
 		$shape.fill.forecolor.rgb = 5880731
 		$shape.fill.OneColorGradient(1, 2, 0.6)
 
 		$shape = $slide.shapes.addShape(1, $xOffsets[$i+1]*72, $yOffset2*72, $vmBoxSize[0]*72, $height2*72)
 		$shape.line.visible = 0
 		if ($height2 -gt 0.4) {
 			$shape.textframe2.textrange.text = "Something Else"
 			$shape.textframe2.textrange.font.size = 24
 		}
 		$shape.fill.forecolor.rgb = 4626167
 		$shape.fill.OneColorGradient(1, 2, 0.6)
 	}
 }
`

