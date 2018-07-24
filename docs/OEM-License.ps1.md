---
Author: alphasun
Publisher: 
Copyright: 
Email: 
Version: 8.1
Encoding: ascii
License: cc0
PoshCode ID: 6292
Published Date: 2017-04-08t19
Archived Date: 2017-04-30t10
---

# oem license - 

## Description

this script is for use with mdt. it was written in order to re-image oem laptops with a clean os and re-use the preinstalled windows product key. the script will attempt to pull the oem product key from wmi and, if available, use slmgr.vbs to install that product key. the same windows version and edition must be used, otherwise this script will fail. if an oem key is not found in wmi, the script attempts to determine the version of windows and use slmgr.vbs to install the generic key for the corresponding version. i have only listed the more common versions of windows.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $ProductKey = $(wmic path softwarelicensingservice get OA3xOriginalProductKey)[2]
 $OSSKU = (Get-WMIObject Win32_OperatingSystem).OperatingSystemSKU
 
 IF ($ProductKey -like "*error*") {
 keys can be activated. 
 
 Reference: http://pastebin.com/SyeWcnKq
 Reference: https://technet.microsoft.com/en-us/library/jj612867.aspx 
 
 These keys are used if Line 2 returns an error. #>
 
 	IF ($OSSKU -eq 101) {
 		cscript C:\Windows\System32\slmgr.vbs -ipk 334NH-RXG76-64THK-C7CKG-D3VPT
 		EXIT }
 
 	ELSEIF ($OSSKU -eq 100) {
 		cscript C:\Windows\System32\slmgr.vbs -ipk Y9NXP-XT8MV-PT9TG-97CT3-9D6TC
 		EXIT }
 
 	ELSEIF ($OSSKU -eq 48) {
 		cscript C:\Windows\System32\slmgr.vbs -ipk GCRJD-8NW9H-F2CDX-CCM8D-9D6T9
 		EXIT }
 
 	ELSEIF ($OSSKU -eq 103) {
 		cscript C:\Windows\System32\slmgr.vbs -ipk GBFNG-2X3TC-8R27F-RMKYB-JK7QT
 		EXIT }
 
 	ELSEIF ($OSSKU -eq 4) {
 		cscript C:\Windows\System32\slmgr.vbs -ipk FHQNR-XYXYC-8PMHT-TV4PH-DRQ3H
 		EXIT }
 
 	ELSE {
 		ECHO "The edition of the operating system either does not match a defined edition in the script, or is not defined in WMI. The script will end."
 		EXIT }
 	}
 
 ELSE {
 
 	cscript C:\Windows\System32\slmgr.vbs -ipk $ProductKey
 	cscript C:\Windows\System32\slmgr.vbs -ato
 	EXIT }
`

