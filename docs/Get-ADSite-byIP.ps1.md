---
Author: nathan linley
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2888
Published Date: 2012-08-04t00
Archived Date: 2017-05-17t13
---

# get-adsite-byip - 

## Description

this script takes an ipv4 address (optionally with subnet mask or mask length) and searches the subnets in active directory from most significant to least significant.  which ever subnet matches the ip address first will be returned in distinguished name format.  if no subnets match, the script will return subnet_not_assigned

## Comments



## Usage



## TODO



## function

`check-ipformat`

## Code

`#
 #
 
 
 	Param(
 		[Parameter(Mandatory=$true,HelpMessage="IP Address")][validatepattern('^\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}$')]$ip,
 		[Parameter(Mandatory=$false,HelpMessage="Netmask")]$nmask,
 		[Parameter(Mandatory=$false,HelpMessage="Mask length")][validaterange(0,32)][int]$nmlength
 	)
 
 function check-ipformat([string]$ip) {
 		if (-not ($ip -match "^\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}$")) {
 			write-error "The Ip address provided: $ip  is not a valid IPv4 address format"
 			return $false
 		} else { 
 			$octetsegments = $ip.split(".")
 			
 			foreach ($octet in $octetsegments) {
 				if ([int]$octet -lt 0 -or [int]$octet -gt 254) {
 					return $false
 				}
 			}
 			return $true 
 		}
 }
 
 
 function get-networkID ([string]$ipaddr, [string]$subnetmask) {
 	
 
 	if (-not (&check-ipformat $ipaddr)) {
 		Write-Host -ForegroundColor "yellow" "IP address provided is not a valid IPv4 address format"
 		Write-Host
 		return $null
 	}
 	
 	if (-not (&check-subnetformat $subnetmask)) {
 		Write-Host -ForegroundColor "yellow" "Subnet mask provided is not a valid format"
 		Write-Host
 		return $null
 	}
 	
 	$ipoctets = $ipaddr.split(".")
 	$subnetoctets = $subnetmask.split(".")
 	$result = ""
 	
 	for ($i = 0; $i -lt 4; $i++) {
 		$result += $ipoctets[$i] -band $subnetoctets[$i]
 		$result += "."
 	}
 	$result = $result.substring(0,$result.length -1)
 	return $result
 	
 }
 	
 function check-subnetformat([string]$subnet) {
 	if (-not ($subnet -match "^\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}$")) {
 		Write-Error "The subnet mask provided does not match IPv4 format"
 		return $false
 	} else {
 		$octetsegments = $subnet.split(".")
 		$foundmostsignficant = $false
 		for ($i = 3; $i -ge 0; $i--) {
 			if ($octetsegments[$i] -ne 0) {
 				if ($foundmostsignificant -eq $true -and $octetsegments[$i] -ne 255) {
 					Write-Error "The subnet mask has an invalid value"
 					return $false
 				} else {
 					if ((255,254,252,248,240,224,192,128) -contains $octetsegments[$i]) {
 						$foundmostsignficant = $true
 					} else {
 						Write-Error "The subnet mask has an invalid value"
 						return $false
 					} 
 					
 				}
 			}
 		}
 		return $true
 	}
 }
 	
 function get-subnetMask-byLength ([int]$length) {
 	if ($length -eq $null -or $length -gt 32 -or $length -lt 0) {
 		Write-Error "get-subnetMask-byLength: Invalid subnet mask length provided.  Please provide a number between 0 and 32"
 		return $null
 	}
 	
 	switch ($length) {
 	 "32" { return "255.255.255.255" }
 	 "31" { return "255.255.255.254" }
 	 "30" { return "255.255.255.252" }
 	 "29" { return "255.255.255.248" }
 	 "28" { return "255.255.255.240" }
 	 "27" { return "255.255.255.224" }
 	 "26" { return "255.255.255.192" }
 	 "25" { return "255.255.255.128" }
 	 "24" { return "255.255.255.0" }
 	 "23" { return "255.255.254.0" }
 	 "22" { return "255.255.252.0" }
 	 "21" { return "255.255.248.0" }
 	 "20" { return "255.255.240.0" }
 	 "19" { return "255.255.224.0" }
 	 "18" { return "255.255.192.0" }
 	 "17" { return "255.255.128.0" }
 	 "16" { return "255.255.0.0" }
 	 "15" { return "255.254.0.0" }
 	 "14" { return "255.252.0.0" }
 	 "13" { return "255.248.0.0" }
 	 "12" { return "255.240.0.0" }
 	 "11" { return "255.224.0.0" }
 	 "10" { return "255.192.0.0" }
 	 "9" { return "255.128.0.0" }
 	 "8" { return "255.0.0.0" }
 	 "7" { return "254.0.0.0"}
 	 "6" { return "252.0.0.0"}
 	 "5" { return "248.0.0.0"}
 	 "4" { return "240.0.0.0"}
 	 "3" { return "224.0.0.0"}
 	 "2" { return "192.0.0.0"}
 	 "1" { return "128.0.0.0"}
 	 "0" { return "0.0.0.0"}
 	
 	}
 	
 }
 
 function get-MaskLength-bySubnet ([string]$subnet) {
 	if ($subnet -eq $null -or (-not(&check-subnetformat $subnet))) {
 		Write-Error "Invalid subnet passed to get-MaskLength-bySubnet in networklib"
 		return $null
 	}
 	
 	switch ($subnet) {
 	"255.255.255.255" {return 32}
 	"255.255.255.254" {return 31}
 	"255.255.255.252" {return 30}
 	"255.255.255.248" {return 29}
 	"255.255.255.240" {return 28}
 	"255.255.255.224" {return 27}
 	"255.255.255.192" {return 26}
 	"255.255.255.128" {return 25}
 	"255.255.255.0"  {return 24}
 	"255.255.254.0"  {return 23}
 	"255.255.252.0"  {return 22}
 	"255.255.248.0"  {return 21}
 	"255.255.240.0" {return 20}
 	"255.255.224.0" {return 19}
 	"255.255.192.0" {return 18}
 	"255.255.128.0" {return 17}
 	"255.255.0.0"  {return 16}
 	"255.254.0.0" {return 15}
 	"255.252.0.0" {return 14}
 	"255.248.0.0" {return 13}
 	"255.240.0.0" {return 12}
 	"255.224.0.0" {return 11}
 	"255.192.0.0" {return 10}
 	"255.128.0.0" {return 9}
 	"255.0.0.0" {return 8}
 	"254.0.0.0" {return 7}
 	"252.0.0.0" {return 6}
 	"248.0.0.0" {return 5}
 	"240.0.0.0"  {return 4}
 	"224.0.0.0" {return 3}
 	"192.0.0.0" {return 2}
 	"128.0.0.0" {return 1}
 	"0.0.0.0"  {return 0}
 	
 	}
 
 }
 	
 	$forest = [System.DirectoryServices.ActiveDirectory.Forest]::GetCurrentForest()
 	$mytopleveldomain = $forest.schema.name
 	$mytopleveldomain = $mytopleveldomain.substring($mytopleveldomain.indexof("DC="))
 	$mytopleveldomain = "LDAP://cn=subnets,cn=sites,cn=configuration," + $mytopleveldomain
 	$de = New-Object directoryservices.DirectoryEntry($mytopleveldomain)
 	$ds = New-Object directoryservices.DirectorySearcher($de)
 	$ds.propertiestoload.add("cn") > $null
 	$ds.propertiestoLoad.add("siteobject") > $null
 	
 	$startMaskLength = 32
 	
 	
 	if ($nmask -ne $null -or $nmlength -ne $null) {
 		if ($nmask -ne $null) {
 			if (-not(&check-subnetformat $nmask)) {
 				Write-Error "Subnet provided is not a valid subnet"
 				exit
 			} else {
 				$startmasklength = &get-MaskLength-bySubnet $nmask
 			}
 		}
 	
 	}
 	
 	for ($i = $startMaskLength; $i -ge 0; $i--) {
 		
 		$mask = &get-subnetMask-byLength $i
 		$netwID = &get-networkID $ip $mask
 		
 		$ds.filter = "(&(objectclass=subnet)(objectcategory=subnet)(cn=" + $netwID + "/" + $i + "))"
 		$fu = $ds.findone()
 		if ($fu -ne $null) {
 			
 			Write-Verbose "Found Subnet in AD at site:"
 			$result = $fu.properties.siteobject
 			return $result
 		}
 		$fu = $null
 	}
 	
 	
 	return "Subnet_NOT_Assigned"
`

