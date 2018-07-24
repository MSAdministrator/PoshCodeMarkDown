---
Author: david
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3212
Published Date: 2012-02-08t11
Archived Date: 2012-02-13t11
---

# hp_plink_iso - 

## Description

plink to map iso on ilo

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $plink = plink -ssh Administrator@$ILOIP -pw $PSWD -auto_store_key_in_cache "set /map1/oemhp_vm1/cddr1 oemhp_image=http://IPADDRESS/ISO.iso"
 $plink = plink -ssh Administrator@$ILOIP -pw $PSWD -auto_store_key_in_cache "set /map1/oemhp_vm1/cddr1 oemhp_boot=connect"
 $plink = plink -ssh Administrator@$ILOIP -pw $PSWD -auto_store_key_in_cache "set /map1/oemhp_vm1/cddr1 oemhp_boot=once"
`

