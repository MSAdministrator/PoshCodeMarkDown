---
Author: dollarunderscore
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6556
Published Date: 2016-10-06t12
Archived Date: 2016-11-29t02
---

# buying groceries - 

## Description

buying groceries with powershell, because why not? ;-)

## Comments



## Usage



## TODO



## function

`connect-mathem`

## Code

`#
 #
 #========================================================================
 #========================================================================
 
 
 function Connect-Mathem
 {
     [cmdletbinding()]
     param(
           [Parameter(Mandatory=$true)]
           [System.Management.Automation.PSCredential] $Credential)
 
     $PostURL = "https://www.mathem.se/Account/Login"
 
     $LoginRequest = @{'UserName'= $Credential.UserName ;'Password' = $Credential.GetNetworkCredential().Password }
 
     $LoginData = Invoke-WebRequest -Uri $PostURL -Body $LoginRequest -Method POST -SessionVariable Global:MathemSession -UseBasicParsing
 }
 
 function Get-FoodItem
 {
     [cmdletbinding()]
     param(
         [Parameter(Mandatory=$True, ValueFromPipelineByPropertyName=$true)]
         [Alias('ProductName')]
         $SearchString)
 
     BEGIN {
         if ($MathemSession -eq $null) {
             Write-Error "You must first connect using the Connect-Mathem cmdlet"
             return
         }
     }
 
     PROCESS {
         $SearchURL = "https://www.mathem.se/WebServices/ProductService.asmx/SearchAndAddResult?searchText="
         $SearchQuery = $SearchString -replace " ","+"
 
         $SearchResults = Invoke-RestMethod -Uri ($SearchURL + $SearchQuery) -WebSession $MathemSession
 
         foreach ($Product in $SearchResults) {
 
             $returnObject = New-Object System.Object
             $returnObject | Add-Member -Type NoteProperty -Name ProductID -Value $Product.ProductID
             $returnObject | Add-Member -Type NoteProperty -Name ProductName -Value $([text.encoding]::utf8.getstring([text.encoding]::default.GetBytes($Product.ProductName)))
             $returnObject | Add-Member -Type NoteProperty -Name Price -Value $Product.Price
             $returnObject | Add-Member -Type NoteProperty -Name PricePerUnit -Value $Product.PricePerUnit
 
             Write-Output $returnObject
         }
     }
 }
 
 function Add-FoodItem
 {
     [cmdletbinding()]
     param(
         [Parameter(Mandatory=$True, ValueFromPipelineByPropertyName=$true)]
         [string] $ProductID,
         [int] $NumberOfItems = 1)
 
     BEGIN {
         if ($MathemSession -eq $null) {
             Write-Error "You must first connect using the Connect-Mathem cmdlet"
             return
         }
     }
 
     PROCESS {
         $AddURL = "https://www.mathem.se/Pages/products/AddToCart.aspx?AddProduct=true&ProductID=$ProductID&noOfFooditem=$NumberOfItems"
 
         Write-Verbose "Adding item $ProductID to the cart..."
 
         $AddItem = Invoke-WebRequest -Uri $AddURL -WebSession $MathemSession -Method Get
 
     }
 
     END { }
 }
 
 
 function Get-FoodBasket
 {
 
     if ($MathemSession -eq $null) {
         Write-Error "You must first connect using the Connect-Mathem cmdlet"
         return
     }
 
     $FoodCartURL = "https://www.mathem.se/ShoppingCart/RenderMiniCart"
 
     $FoodBasket = Invoke-RestMethod -Uri $FoodCartURL -WebSession $MathemSession
     
     $ItemsFoundInBasket = $FoodBasket.div.div.form.ul.li
 
     $ProductsInBasket = @()
 
     if ($ItemsFoundInBasket -ne $null) {
         $ProductsInBasket += $ItemsFoundInBasket
     }
 
     if ($ProductsInBasket.count -gt 0) {
         foreach ($Product in $ProductsInBasket) {
             $returnObject = New-Object System.Object
             $returnObject | Add-Member -Type NoteProperty -Name ProductID -Value $Product."data-highlight-id"
             $returnObject | Add-Member -Type NoteProperty -Name ProductLink -Value $($Product.table.tbody.tr.td | select -Skip 1 -First 1 | % { $_.a.href })
             $returnObject | Add-Member -Type NoteProperty -Name TotalPrice -Value $($Product.table.tbody.tr.td | select -Skip 2 -First 1 | % { $_.strong })
             $returnObject | Add-Member -Type NoteProperty -Name NumberOfItems -Value $Product.table.tbody.tr.td.div.input.value
 
             Write-Output $returnObject
         }
     }
 }
 
 function Remove-FoodItem
 {
     [cmdletbinding()]
     param(
         [Parameter(Mandatory=$True, ValueFromPipelineByPropertyName=$true)]
         [string] $ProductID)
 
     begin {
         if ($MathemSession -eq $null) {
             Write-Error "You must first connect using the Connect-Mathem cmdlet"
             return
         }
     }
 
     PROCESS {
         $DeleteURL = "https://www.mathem.se/ShoppingCart/Delete/$ProductID"
 
         Write-Verbose "Removing item $ProductID from cart..."
 
         $DeleteItem = Invoke-WebRequest -Uri $DeleteURL -WebSession $MathemSession -Method Get
     }
 
     END { }
 }
 
 function Get-FoodItemMostBought
 {
     if ($MathemSession -eq $null) {
         Write-Error "You must first connect using the Connect-Mathem cmdlet"
         return
     }
 
 
     $MostBoughtFoodItemsSiteFile = Invoke-WebRequest -Uri $MostBoughtURL -WebSession $MathemSession -UseBasicParsing -OutFile .\tempfile.site
     $MostBoughtFoodItemsSite = Get-Content .\tempfile.site -Encoding UTF8
     Remove-Item .\tempfile.site -Force
     $MostBoughtFoodItems = $MostBoughtFoodItemsSite.split("`n") | Select-String -Pattern "class=`"prodTitle`"" | % { (($_ -split ">")[1] -split "</a")[0] } | select -Skip 1 | Sort-Object -Unique
     
     
     foreach ($Item in $MostBoughtFoodItems) {
 
         $returnObject = New-Object System.Object
         $returnObject | Add-Member -Type NoteProperty -Name ProductName -Value $Item
 
         Write-Output $returnObject
     }
 }
 
 
 function Add-MatHemBonusCode
 {
     [cmdletbinding()]
     param(
         [Parameter(Mandatory=$True, ValueFromPipelineByPropertyName=$true)]
         [string] $BonusCode)
 
     begin {
         if ($MathemSession -eq $null) {
             Write-Error "You must first connect using the Connect-Mathem cmdlet"
             return
         }
     }
 
     PROCESS {
         $AddCodeURL = "https://www.mathem.se/checkout/addbonuscode/$BonusCode"
 
         Write-Verbose "Adding bonus code $BonusCode"
 
         $BonusCodeResponse = Invoke-RestMethod -Uri $AddCodeURL -WebSession $MathemSession -Method Get
 
         if ($BonusCodeResponse.Error) {
             throw "The request failed with message: $($BonusCodeResponse.Message)"
         }
         else {
             $BonusCodeResponse.Message
         }
     }
 
     END { }
 }
`

