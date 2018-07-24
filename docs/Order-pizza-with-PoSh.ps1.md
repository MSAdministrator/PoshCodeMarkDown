---
Author: dollarunderscore
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 6016
Published Date: 2016-09-15t20
Archived Date: 2016-05-17t13
---

# order pizza with posh - 

## Description

this code is from a web scrape guide i’ve written on my blog.

## Comments



## Usage



## TODO



## function

`connect-onlinepizza`

## Code

`#
 #
 function Connect-OnlinePizza
 {
     [cmdletbinding()]
     param(
           [Parameter(Mandatory=$True)]
           [System.Management.Automation.PSCredential] $Credential)
 
     $Username = $Credential.UserName
     $Password = $Credential.GetNetworkCredential().Password
 
     $Request = @{'username' = $Username
                  'password'= $Password
                  'action'= 'loggain'}
 
     Invoke-WebRequest -Uri "https://onlinepizza.se/loggain" -Method Post -Body $Request -SessionVariable Global:OnlinePizzaSession -OutFile .\dump.htm
 
     $LoggedIn = Select-String -Path .\dump.htm -Pattern "inloggad som $Username" -Quiet
 
     Remove-Item .\dump.htm -Force -Confirm:$false -ErrorAction SilentlyContinue
 
     if ($LoggedIn) {
         Write-Verbose "You are now logged in!"
     }
     else {
         Write-Error "Login failed!"
     }
 }
 
 function Get-MyOnlinePizzaAccountInfo
 {
     [cmdletbinding()]
     param()
 
     BEGIN {
         if ($OnlinePizzaSession -eq $null) {
             Write-Error "You must first connect using the Connect-OnlinePizza cmdlet"
             break
         }
     }
 
     PROCESS {
 
         Invoke-WebRequest -Uri "http://onlinepizza.se/?view=andraKonto" -Method Get -WebSession $Global:OnlinePizzaSession -OutFile .\dump.htm
 
         $AccountInfo = Get-Content .\dump.htm -Encoding UTF8
 
         Remove-Item .\dump.htm -Force -Confirm:$false -ErrorAction SilentlyContinue
 
         $Username = ((($AccountInfo | Select-String -Pattern "name=username id=username") -split "value=`"")[1] -split "`" />")[0]
         $AccountHolderName = ((($AccountInfo | Select-String -Pattern "name=namn id=namn") -split "value=`"")[1] -split "`"/>")[0]
         $AccountHolderMail = ((($AccountInfo | Select-String -Pattern "name=epost id=epost") -split "value=`"")[1] -split "`"/>")[0]
         $AccountHolderStreet = ((($AccountInfo | Select-String -Pattern "name=adress1 id=adress1") -split "value=`"")[1] -split "`"/>")[0]
         $AccountHolderPostalCode = ((($AccountInfo | Select-String -Pattern "name=postnummer id=postnummer") -split "value=`"")[1] -split "`"/>")[0]
         $AccountHolderPhone = ((($AccountInfo | Select-String -Pattern "name=telefon id=telefon") -split "value=`"")[1] -split "`"/>")[0]
 
         $returnObject = New-Object System.Object
         $returnObject | Add-Member -Type NoteProperty -Name Username -Value $Username
         $returnObject | Add-Member -Type NoteProperty -Name Name -Value $AccountHolderName
         $returnObject | Add-Member -Type NoteProperty -Name Email -Value $AccountHolderMail
         $returnObject | Add-Member -Type NoteProperty -Name Address -Value $AccountHolderStreet
         $returnObject | Add-Member -Type NoteProperty -Name PostalCode -Value $AccountHolderPostalCode
         $returnObject | Add-Member -Type NoteProperty -Name Phone -Value $AccountHolderPhone
 
         Write-Output $returnObject
 
     }
 
     END { }
 }
 
 function Get-PizzaRestaurant
 {
     [cmdletbinding()]
     param(
           [Parameter(Mandatory=$True,ValueFromPipelineByPropertyName=$true)]
           [int] $PostalCode)
 
     BEGIN {
         if ($OnlinePizzaSession -eq $null) {
             Write-Error "You must first connect using the Connect-OnlinePizza cmdlet"
             break
         }
     }
 
     PROCESS {
 
         Invoke-WebRequest -Uri "http://onlinepizza.se/postnummer/$PostalCode" -Method Get -WebSession $Global:OnlinePizzaSession -OutFile .\dump.htm
 
         $ResturantList = ((Get-Content .\dump.htm) -join "`n") -split "<UL>" | select -Skip 1
 
         Remove-Item .\dump.htm -Force -Confirm:$false -ErrorAction SilentlyContinue
 
         foreach ($Restaurant in $ResturantList) {
 
             $RestaurantName = (($Restaurant -split "<h4>")[1] -split "</h4>")[0]
 
             if ($RestaurantName -eq '') {
                 Continue
             }
 
             $RestaurantStreet = (($Restaurant -split "<address>")[1] -split "</address>")[0]
             $OpeningHoursDelivery = ((($Restaurant -split "Utkörning:</strong><br />")[1] -split "<br />")[0]).Trim()
             $OpeningHoursTakeAway = ((($Restaurant -split "Avhämtning:</strong><br />")[1] -split "<br />")[0]).Trim()
             $RestaurantLink = ((($Restaurant -split "meny")[0] -split "href=`"")[1] -split "`"")[0]
 
             $returnObject = New-Object System.Object
             $returnObject | Add-Member -Type NoteProperty -Name RestaurantName -Value $RestaurantName
             $returnObject | Add-Member -Type NoteProperty -Name RestaurantStreet -Value $RestaurantStreet
             $returnObject | Add-Member -Type NoteProperty -Name OpeningHoursDelivery -Value $OpeningHoursDelivery
             $returnObject | Add-Member -Type NoteProperty -Name OpeningHoursTakeAway -Value $OpeningHoursTakeAway
             $returnObject | Add-Member -Type NoteProperty -Name RestaurantLink -Value $RestaurantLink
 
             Write-Output $returnObject
 
             Remove-Variable RestaurantName, RestaurantStreet, OpeningHoursDelivery, OpeningHoursTakeAway, RestaurantLink -ErrorAction SilentlyContinue
         }
     }
 
     END { }
 }
`

