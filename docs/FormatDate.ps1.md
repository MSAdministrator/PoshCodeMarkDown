---
Author: ferenc toth
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5129
Published Date: 2014-04-30t11
Archived Date: 2014-05-02t00
---

# formatdate - 

## Description

get input date string or get t-n or t+n dates in the specified format.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 Function FormatDate {
 <#
 .SYNOPSIS
 Get input date string or get T-n or T+n dates in the specified format.
 .DESCRIPTION
 Author: Ferenc Toth
 Date:	20/03/2014
 .INPUTS
 .PARAMETER Date
 Date string in format YYYY/MM/DD or MM/DD/YYYY - Default is today
 .PARAMETER Format
 Output date format - Default is yyyMMdd
 
 More info about date format in Powershell: http://technet.microsoft.com/en-us/library/ee692801.aspx
 .PARAMETER Offset
 Offset (+/-) days
 .PARAMETER AllDays
 Do not skip weekends. By default you have business days (Mon-Fri) only.
 .LINK
 
 .EXAMPLE
 C:\PS>FormatDate
 Returns with today's date in YYYYMMDD format
 .EXAMPLE
 C:\PS>FormatDate -Offset 1
 Returns with T+1 date in YYYYMMDD format
 .EXAMPLE
 C:\PS>FormatDate -Offset -2
 Returns with T-2 date in YYYYMMDD format
 .EXAMPLE
 C:\PS>FormatDate -Date "2014/03/02" -Format ddMMyy
 Returns with 020314
 
 More info about date format in Powershell: http://technet.microsoft.com/en-us/library/ee692801.aspx
 #>
 
     [CmdletBinding()]
     [OutputType([Object])]
     param (
         [parameter(Mandatory=$false)]
 		[DateTime] $InputDate=(Get-Date),
 		[parameter(Mandatory=$false)]
 		[string] $Format="yyyMMdd",
 		[parameter(Mandatory=$false)]
         [int] $Offset=0,
 		[parameter()]
         [switch] $AllDays=$False
     )
     
     process {
 		if ($Offset -lt 0) {
 			$Private:NEWDATE=$(Get-Date -Date $InputDate)
 			for($i=$Offset;$i -lt 0;$i++) {
 				$NEWDATE=$($NEWDATE).AddDays(-1)
 				if(0 -eq $($NEWDATE.DayOfWeek) -and !$AllDays) {
 					$NEWDATE=$($NEWDATE).AddDays(-2)
 				} elseif(6 -eq $($NEWDATE.DayOfWeek) -and !$AllDays) {
 					$NEWDATE=$($NEWDATE).AddDays(-1)
 				}
 			}
 			$($NEWDATE).ToString($Format)
 		} elseif ($Offset -gt 0) {
 			$Private:NEWDATE=$(Get-Date -Date $InputDate)
 			for($i=$Offset;$i -gt 0;$i--) {
 				$NEWDATE=$($NEWDATE).AddDays(1)
 				if(6 -eq $($NEWDATE.DayOfWeek) -and !$AllDays) {
 					$NEWDATE=$($NEWDATE).AddDays(2)
 				}
 			}
 			$($NEWDATE).ToString($Format)
 		} else {
 			Get-Date -Date $InputDate -Format $Format
 		}
 	}
 }
 
 set-alias fd FormatDate
 Export-ModuleMember -alias fd
 Export-ModuleMember -function FormatDate
`

