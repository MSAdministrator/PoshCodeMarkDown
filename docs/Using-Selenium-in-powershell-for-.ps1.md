---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 2.47.0
Encoding: ascii
License: cc0
PoshCode ID: 5970
Published Date: 
Archived Date: 2016-05-17t13
---

#  - 

## Description

using selenium in powershell for scraping data from ruckus zonedirectors

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 ################################
  $result = "C:\ProgramData\Paessler\PRTG Network Monitor\Logs (Sensors)\result.txt"
 
  $userName = "REDACTED!"
  $passWord = "REDACTED!"
 
  $zoneDirectors = "https://zonedirector1.yourcompany.org","https://zonedirector2.yourcompany.org"
 
  $url1 = "/admin/login.jsp"
   $url3 = "/admin/dashboard.jsp"
   $urlWLANS = "/admin/mon_wlans.jsp"
 
  
 ################################
 
  $dllroot = "C:\Selenium.WebDriver.2.47.0\lib\net40\selenium-dotnet-2.47.0\net35"
 
  Add-Type -Path  (join-path $dllroot Selenium.WebDriverBackedSelenium.dll)
  Add-Type -Path (join-path $dllRoot ThoughtWorks.Selenium.Core.dll)
  Add-Type -Path (join-path $dllRoot WebDriver.dll)
  Add-Type -Path (join-path $dllRoot WebDriver.Support.dll)
  
  $driver = New-Object OpenQA.Selenium.Chrome.ChromeDriver
 
  function waitForLoad{
     param(
         $elementID
     )
     try {
         $loadStatus = $true
         $driver.FindElementById($elementID)
         }
     catch
      [OpenQA.Selenium.NoSuchElementException]
      {
         $loadStatus = $false
      }
      if (($driver.FindElementById($elementID)).tostring().length -lt 2){$loadStatus = $false; write-host "almost"}
      write-host "$elementID $loadStatus"
      return $loadStatus
  }
  
  
  
 
  
  
  function getStats{
     [cmdletbinding()]
     param([string]$zd)
     write-host "Working on $zd"
     
      $driver.Navigate().GoToUrl($zd)
     ($driver.FindElementById("username")).sendKeys($userName)
     ($driver.FindElementByName("password")).sendKeys($passWord)
     ($driver.FindElementByName("ok")).Click()
    
     while((waitForLoad "num-client") -eq $false){start-sleep -m 500}
     while($driver.FindElementById("num-client").text.toString().length -lt 1){start-sleep -m 200}
     
     [int]$authClients = $driver.FindElementById("num-client").text
     write-host "authd = $authClients"
     
     $driver.Navigate().GoToUrl($zd + $urlWLANS)
 
     while((waitForLoad "wlansummary") -eq $false){start-sleep -m 500}
     
 
         param ([string]$wlanName)
         $counted = ($driver.FindElementById("wlansummary").text -split ("\r\n") | select-string $wlanName).tostring().Split(" ") | select -last 1
         return [int]$counted
         }
     
     while(($driver.FindElementById("wlansummary").text -split ("\r\n") | select-string "Net5ghz").count -lt 1){start-sleep -m 200}
     $5gClients = produceClients("Net5ghz")
     write-host "5G = $5gClients"
     $byodClients = produceClients("NetBYOD")
     write-host "BYOD = $byodclients"
`

