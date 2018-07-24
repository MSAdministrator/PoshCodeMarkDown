---
Author: vulcanx
Publisher: 
Copyright: 
Email: 
Version: 1.1
Encoding: ascii
License: cc0
PoshCode ID: 5145
Published Date: 2015-05-05t10
Archived Date: 2015-01-31t20
---

# hardware inventory - 

## Description

hardware inventory script to run on windows servers and output the json string to a couchdb database

## Comments



## Usage



## TODO



## script

`new-jsonproperty`

## Code

`#
 #
 
 
 
 
 Write-Host "
 ----------------------------------------------
  _   _ _____      _                 _ 
 | \ | |_   _|    / \   ___ ___  ___| |_
 |  \| | | |     / _ \ / __/ __|/ _ \ __|
 | |\  | | |    / ___ \\__ \__ \  __/ |_
 |_| \_| |_|   /_/   \_\___/___/\___|\__|
                                       
  ___                      _                 
 |_ _|_ ____   _____ _ __ | |_ ___  _ __ _   _
  | || '_ \ \ / / _ \ '_ \| __/ _ \| '__| | | |
  | || | | \ V /  __/ | | | || (_) | |  | |_| |
 |___|_| |_|\_/ \___|_| |_|\__\___/|_|   \__, |
                                         |___/
  
             by VulcanX
             2011-09-29
                 v1.1
 ----------------------------------------------
 "
 
 
 Filter ConvertTo-JSON {
     Function New-JSONProperty ($name, $value) {
 @"
     "$name": $value
 "@
     }
 
     $targetObject = $_
     $jsonProperties = @()
     $properties = $_ | Get-Member -MemberType *property
 
     ForEach ($property in $properties) {
         #"$($property.Name)=$($targetObject.$($property.Name)) [$($($targetObject.$($property.Name)).GetType())]"
         #(($targetObject.($property.Name)).GetType()).Name
 
         $value = $targetObject.($property.Name)
         $dataType = (($targetObject.($property.Name)).GetType()).Name
 
         switch -regex ($dataType) {
             'String'  {$jsonProperties += New-JSONProperty $property.Name "`"$value`""}
             'Int32|Double' {$jsonProperties += New-JSONProperty $property.Name $value}
             'Object\[\]' {
                 #$jsonProperties += "`t`"$($property.Name)`": [$($($targetObject.($property.Name)) -join ',')]"
                 $str = "`t`"$($property.Name)`": ["
                 
                 $count = $targetObject.($property.Name).Count
                 For($idx=0; $idx -lt $count; $idx++) {
                     $v = $targetObject.($property.Name)[$idx]
                     
                     switch -regex ($v.GetType()) {
                         'String' {$v="`"$v`""}
                     }
                     
                     if($idx+1 -lt $count) {
                         $comma = ","
                     } else {
                         $comma = ""
                     }
                     
                     $str += "$v$($comma)"
                 }
                 
                 
                 $jsonProperties += "$str]"
             }
             default {$_}
         }
     }
 
     "{`r`n $($jsonProperties -join ",`r`n") `r`n}"
 }
 
 
 $HOSTNAME1 = Get-WmiObject win32_computersystem | Select-Object -ExpandProperty Name
 $HOSTNAME2 = Get-WmiObject win32_computersystem | Select-Object -ExpandProperty Domain
 $HOSTNAME = "$HOSTNAME1.$HOSTNAME2"
 $OS = Get-WmiObject win32_operatingsystem | Select-Object -ExpandProperty Caption
 $OS_RELEASE = Get-WmiObject win32_operatingsystem | Select-Object -ExpandProperty Version
 $IP_ADDRESS_1 = Get-WmiObject win32_NetworkAdapterConfiguration | Where { $_.IPAddress -and $_.IPAddress -notlike ':' } | Select -ExpandProperty IPAddress
 $MAC_ADDRESS_1 = Get-WmiObject win32_networkadapter | Where { $_.PhysicalAdapter -eq $TRUE -and $_.MacAddress -ne $null } | Select-Object -ExpandProperty MACAddress
 $MEMORY = Get-WmiObject win32_computersystem | Select-Object -ExpandProperty TotalPhysicalMemory
 $BIOS_VENDOR = Get-WmiObject win32_bios | Select-Object -ExpandProperty Manufacturer
 $BIOS_VERSION = Get-WmiObject win32_bios | Select-Object -ExpandProperty SMBIOSBIOSVersion
 $BIOS_RELEASE_DATE = Get-WmiObject win32_bios | Select-Object @{label='Release Date';expression={$_.ConvertToDateTime($_.releasedate).ToShortDateString()}} | Select-Object -ExpandProperty 'Release Date'
 $SYSTEM_MANUFACTURER = Get-WmiObject win32_computersystemproduct | Select-Object -ExpandProperty Vendor
 $SYSTEM_PRODUCT_NAME = Get-WmiObject win32_computersystemproduct | Select-Object -ExpandProperty Name
 $SYSTEM_VERSION = Get-WmiObject win32_computersystemproduct | Select-Object -ExpandProperty Version
 $SYSTEM_SERIAL_NUMBER = Get-WmiObject win32_computersystemproduct | Select-Object -ExpandProperty IdentifyingNumber
 $SYSTEM_UUID = Get-WmiObject win32_computersystemproduct | Select-Object -ExpandProperty UUID
 $BASEBOARD_MANUFACTURER = Get-WmiObject win32_baseboard | Select-Object -ExpandProperty Manufacturer
 $BASEBOARD_PRODUCT_NAME = Get-WmiObject win32_baseboard | Select-Object -ExpandProperty Product
 $BASEBOARD_VERSION = Get-WmiObject win32_baseboard | Select-Object -ExpandProperty Version
 $BASEBOARD_SERIAL_NUMBER = Get-WmiObject win32_baseboard | Select-Object -ExpandProperty SerialNumber
 $BASEBOARD_ASSET_TAG = Get-WmiObject win32_baseboard | Select-Object -ExpandProperty Tag
 $CHASSIS_MANUFACTURER = Get-WmiObject win32_systemenclosure | Select-Object -ExpandProperty Manufacturer
 $CHASSIS_TYPE = Get-WmiObject win32_systemenclosure | Select-Object -ExpandProperty ChassisTypes
 $CHASSIS_VERSION = Get-WmiObject win32_systemenclosure | Select-Object -ExpandProperty Version
 $CHASSIS_SERIAL_NUMBER = Get-WmiObject win32_systemenclosure | Select-Object -ExpandProperty SerialNumber
 $CHASSIS_ASSET_TAG = Get-WmiObject win32_systemenclosure | Select-Object -ExpandProperty SMBIOSAssetTag
 {
 	12	{"Pentium Pro"}
 	132	{"AMD Opteron"}
 	176 	{"Pentium 3 Xeon"}
 	179	{"Intel Xeon"}
 	181	{"Intel Xeon MP"}
 	Default {$PROCESSOR_FAMILY1}
 }
 $PROCESSOR_MANUFACTURER = Get-WmiObject win32_processor | Select-Object -ExpandProperty Manufacturer -First 1
 $PROCESSOR_VERSION = Get-WmiObject win32_processor | Select-Object -ExpandProperty Name -First 1
 {
 	0	{"x86"}
 	1	{"MIPS"}
 	2	{"Alpha"}
 	3	{"PowerPC"}
 	6	{"Itanium"}
 	9	{"x64"}
 	Default {$PROCESSOR_ARCH1}
 }
 $PROCESSOR_FREQUENCY = Get-WmiObject win32_processor | Select-Object -ExpandProperty MaxClockSpeed -First 1
 
 $RESULTS = (New-Object PSObject |
 add-member -pass noteproperty HOSTNAME "$HOSTNAME" |
 add-member -pass noteproperty OS "$OS" |
 add-member -pass noteproperty OS-RELEASE "$OS_RELEASE" |
 add-member -pass noteproperty IP-ADDRESS "$IP_ADDRESS" |
 add-member -pass noteproperty MAC-ADDRESS "$MAC_ADDRESS" |
 add-member -pass noteproperty MEMORY "$MEMORY" |
 add-member -pass noteproperty BIOS-VENDOR "$BIOS_VENDOR" |
 add-member -pass noteproperty BIOS-VERSION "$BIOS_VERSION" |
 add-member -pass noteproperty BIOS-RELEASE-DATE "$BIOS_RELEASE_DATE" |
 add-member -pass noteproperty SYSTEM-MANUFACTURER "$SYSTEM_MANUFACTURER" |
 add-member -pass noteproperty SYSTEM-PRODUCT-NAME "$SYSTEM_PRODUCT_NAME" |
 add-member -pass noteproperty SYSTEM-VERSION "$SYSTEM_VERSION" |
 add-member -pass noteproperty SYSTEM-SERIAL-NUMBER "$SYSTEM_SERIAL_NUMBER" |
 add-member -pass noteproperty SYSTEM-UUID "$SYSTEM_UUID" |
 add-member -pass noteproperty BASEBOARD-MANUFACTURER "$BASEBOARD_MANUFACTURER" |
 add-member -pass noteproperty BASEBOARD-PRODUCT-NAME "$BASEBOARD_PRODUCT_NAME" |
 add-member -pass noteproperty BASEBOARD-VERSION "$BASEBOARD_VERSION" |
 add-member -pass noteproperty BASEBOARD-SERIAL-NUMBER "$BASEBOARD_SERIAL_NUMBER" |
 add-member -pass noteproperty BASEBOARD-ASSET-TAG "$BASEBOARD_ASSET_TAG" |
 add-member -pass noteproperty CHASSIS-MANUFACTURER "$CHASSIS_MANUFACTURER" |
 add-member -pass noteproperty CHASSIS-TYPE "$CHASSIS_TYPE" |
 add-member -pass noteproperty CHASSIS-VERSION "$CHASSIS_VERSION" |
 add-member -pass noteproperty CHASSIS-SERIAL-NUMBER "$CHASSIS_SERIAL_NUMBER" |
 add-member -pass noteproperty CHASSIS-ASSET-TAG "$CHASSIS_ASSET_TAG" |
 add-member -pass noteproperty PROCESSOR-FAMILY "$PROCESSOR_FAMILY" |
 add-member -pass noteproperty PROCESSOR-MANUFACTURER "$PROCESSOR_MANUFACTURER" |
 add-member -pass noteproperty PROCESSOR-VERSION "$PROCESSOR_VERSION" |
 add-member -pass noteproperty PROCESSOR-ARCH "$PROCESSOR_ARCH" |
 
 echo $RESULTS
 
 $username = Read-Host "CouchDB Username"
 $password = Read-Host "CouchDB Password"
 
 [System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true} ;
 
 $auth = [System.Text.Encoding]::UTF8.GetBytes("$username" + ":" + "$password")
 $base64 = [System.Convert]::ToBase64String($auth)
 $authorization = "Authorization: Basic " + $base64
 $request.Headers.Add($authorization)
 
 $request.Credentials = New-Object System.Net.NetworkCredential($username, $password)
 $data = [Text.Encoding]::UTF8.GetBytes($RESULTS)
 
 $request.ContentLength = $data.Length
 $request.Method = "PUT"
 
 
 $put = new-object IO.StreamWriter $request.GetRequestStream()
 $put.Write($data,0,$data.Length)
 $put.Flush()
 $put.Close()
`

