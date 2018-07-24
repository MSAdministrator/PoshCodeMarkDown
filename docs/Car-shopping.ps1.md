---
Author: zefram
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5493
Published Date: 2014-10-08t15
Archived Date: 2014-10-11t09
---

# car shopping. - 

## Description

i’m sick of searching for cars on ksl manually.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 #
 #
 
 [string]$KeywordSearch = 'civic+si'
 [string]$YearStart = '2010'
 [string]$YearEnd = '2014'
 [string]$PriceFrom = '15000'
 [string]$PriceTo = '20000'
 
 Try
 {
 	Write-Verbose "Requesting URI"
 	$Site = "http://www.ksl.com/auto/search/index?o_facetClicked=true&o_facetValue=Coupe&o_facetKey=body&resetPage=true&keyword=$KeywordSearch&yearFrom=$YearStart&yearTo=$YearEnd&priceFrom=$PriceFrom&priceTo=$PriceTo&mileageFrom=&mileageTo=&zip=&miles=0&body[]=$BodyStyle"
 	$Request = Invoke-WebRequest -Uri $Site -DisableKeepAlive -Method Get
 	
 	Write-Verbose "Filtering cars listing from KSL"
 	$listing = 	$Request.ParsedHtml.getElementsByTagName('div') | ? {$_.className -eq 'srp-listing-body-right'}
 	
 	Write-Verbose "Building cars object"
 	$cars = @()
 	
 	Foreach ($item in $listing.outerText)
 	{
 		$details = $item.Trim().Split("`n")
 		
 		$moredetails = $details[2].Split('|')
 
 		$car = New-Object -TypeName PSObject
 		$car | Add-Member -Type NoteProperty -Name 'Title' -Value $details[0]
 		$car | Add-Member -Type NoteProperty -Name 'Price' -Value $details[1]
 		$car | Add-Member -Type NoteProperty -Name 'Mileage' -Value $moredetails[0].Trim().Replace(' Miles','')
 		$car | Add-Member -Type NoteProperty -Name 'Location' -Value  $moredetails[1].Trim()
 		$car | Add-Member -Type NoteProperty -Name 'DaysListed' -Value $moredetails[2].Trim().Replace(' Days','')
 		$car | Add-Member -Type NoteProperty -Name 'Description' -Value $details[3]
 		
 		$cars += $car
 	}
 	
 	$cars = $cars | Sort -Property DaysListed
 	$cars
 }
 Catch
 {
 	Write-Warning "Something screwed up`: $_"
 	Exit
 }
`

