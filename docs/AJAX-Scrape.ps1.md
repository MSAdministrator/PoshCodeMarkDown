---
Author: foureight84
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3144
Published Date: 2012-01-05t10
Archived Date: 2017-03-10t08
---

# ajax scrape - 

## Description

scrape ajax driven websites example using watin and htmlagilitypack.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 $rootDir = "C:\Users\khtruong\Desktop\android review scrape"
 $WatiNPath = "$rootDir\WatiN.Core.dll"
 $HtmlAgilityPath = "$rootDir\HtmlAgilityPack.dll"
 
 [reflection.assembly]::loadfrom( $WatiNPath )
 [reflection.assembly]::loadfrom( $HtmlAgilityPath )
 
 $ie = New-Object Watin.Core.IE
 
 $packages = @("com.glu.android.gunbros_free", "com.rovio.angrybirds")
 
 $global:reviews = @()
 
 foreach($package in $packages){
 	$ie.Goto("https://market.android.com/details?id=$package")
 	$ie.WaitForComplete(300)
 	
 	$($ie.Links | ?{$_.ClassName -eq "tabBarLink"}).Click()
 
 	$($($ie.Divs  | ?{$_.ClassName -eq "reviews-sort-menu-container goog-inline-block"}).Divs | ?{$_.ClassName -eq "goog-inline-block selected-option"}).ClickNoWait()
 
 	$($($($ie.Divs | ?{$_.ClassName -eq "reviews-menu"}).Divs | ?{$_.ClassName -eq "goog-menuitem-content"})[0]).ClickNoWait()
 
 	$lastPage = $false
 	$nextButton = $($ie.Divs | ?{$_.ClassName -eq "num-pagination-page-button num-pagination-next goog-inline-block"})
 
 	$count = 1
 	
 	while($count -lt 49){
 		write-host $count
 		$nextButton.Click()
 		Sleep 1
 		$count++
 	}
 	
 	$result = $ie.Html
 
 	$doc = New-Object HtmlAgilityPack.HtmlDocument 
 
 	$doc.LoadHtml($result)
 
 	$reviewSize = $($doc.DocumentNode.SelectNodes("//div[@class='doc-review']")).length
 
 	$reviews += @(for($counter = 0; $counter -lt $reviewSize; $counter++){
 		if($($($($doc.DocumentNode.SelectNodes("//div[@class='doc-review']"))[-1]).ChildNodes[3].ChildNodes | %{$_.Attributes | ?{$_.Name -eq "href"}}).Value -ne $null){
 			Write-Host "($counter / $reviewSize)" -fore Yellow
 			$PackageName = $($($($doc.DocumentNode.SelectNodes("//div[@class='doc-review']"))[$counter]).ChildNodes[3].ChildNodes | %{$_.Attributes | ?{$_.Name -eq "href"}}).Value.Split("=&")[1]
 			$ReviewID = $($($($doc.DocumentNode.SelectNodes("//div[@class='doc-review']"))[$counter]).ChildNodes[3].ChildNodes | %{$_.Attributes | ?{$_.Name -eq "href"}}).Value.Split("=&")[-1]
 			Write-Host "$ReviewID"
 		}
 
 		if($($($doc.DocumentNode.SelectNodes("//div[@class='doc-review']"))[$counter]).ChildNodes[0].InnerText -ne $null){
 			$Author = $($($doc.DocumentNode.SelectNodes("//div[@class='doc-review']"))[$counter]).ChildNodes[0].InnerText
 		}
 		else{
 			$Author = "Unknown"
 		}
 		
 		if($($($($doc.DocumentNode.SelectNodes("//div[@class='doc-review']"))[$counter]).ChildNodes[1].InnerText).Replace(" on ","").Trim() -ne $null){
 			$Date = $($($($doc.DocumentNode.SelectNodes("//div[@class='doc-review']"))[$counter]).ChildNodes[1].InnerText).Replace(" on ","").Trim()
 		}
 		else{
 			$Date = "Unknown"
 		}
 
 		if($($($doc.DocumentNode.SelectNodes("//div[@class='doc-review']"))[$counter]).ChildNodes[2].InnerText -like "*with*"){
 			$Handset = $($($($($doc.DocumentNode.SelectNodes("//div[@class='doc-review']"))[$counter]).ChildNodes[2].InnerText).Trim().replace("with","|").Split("|")[0]).Replace("(","").trim()
 		}
 		else{
 			$Handset = "Unknown"
 		}
 
 		if($($($($doc.DocumentNode.SelectNodes("//div[@class='doc-review']"))[$counter]).ChildNodes[2].InnerText).Trim().Split(" ")[-1].replace(")","").Trim() -ne $null){
 			$Version = $($($($doc.DocumentNode.SelectNodes("//div[@class='doc-review']"))[$counter]).ChildNodes[2].InnerText).Trim().Split(" ")[-1].replace(")","").Trim()
 		}
 		else{
 			$Version = "Unknown"
 		}
 
 		if($($($($($($($doc.DocumentNode.SelectNodes("//div[@class='doc-review']"))[$counter]).ChildNodes[4]).ChildNodes) | %{$_.Attributes | ?{$_.Name -eq "Title"}}).Value) -ne $null){
 			$Rating = [Int]$($($($($($($doc.DocumentNode.SelectNodes("//div[@class='doc-review']"))[$counter]).ChildNodes[4]).ChildNodes) | %{$_.Attributes | ?{$_.Name -eq "Title"}}).Value).Split(" ")[1]
 			
 			if($Rating -lt 3){
 				$Flag = "Critical"
 			}
 			else{
 				$Flag = ""
 			}
 			
 		}
 		else{
 			$Rating = "Unknown"
 		}
 
 		if($($($($doc.DocumentNode.SelectNodes("//div[@class='doc-review']"))[$counter]).ChildNodes[4].InnerText) -ne $null){
 			$Title = $($($($doc.DocumentNode.SelectNodes("//div[@class='doc-review']"))[$counter]).ChildNodes[4].InnerText)
 		}
 		else{
 			$Title = "Review title not given."
 		}
 
 		if($($($($doc.DocumentNode.SelectNodes("//div[@class='doc-review']"))[$counter]).ChildNodes[5].InnerText) -ne "&nbsp;"){
 			$Review = $($($($doc.DocumentNode.SelectNodes("//div[@class='doc-review']"))[$counter]).ChildNodes[5].InnerText)
 		}
 		else{
 			$Review = "User did not write a review."
 		}
 		
 		New-Object psobject -Property @{
 			PackageName = $PackageName
 			ReviewID = $ReviewID
 			Author = $Author
 			Date = $Date
 			Handset = $Handset
 			Version = $Version
 			Rating = $Rating
 			Title = $Title
 			Review = $Review
 			Flag = $Flag
 		}
 	})
 }
`

