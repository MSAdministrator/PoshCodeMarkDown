---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4560
Published Date: 2013-10-26t16
Archived Date: 2013-11-01t01
---

# check new gmail - 

## Description

sometimes i have to deal with bash, so…

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 #!/bin/bash
 
 num="\033[1;36m"
 end="\033[0m"
 
 read -p "Enter email without '@gmail.com': " email
 read -p "Enter password of email: " pass
 
 atom=`wget -qO - https://$email:$pass@mail.google.com/mail/feed/atom \
    --secure-protocol=TLSv1 -T 3 -t 1 --no-check-certificat | grep \
    fullcount | sed -e 's/<fullcount>\(.*\)<\/fullcount>/\1/'`
 
 echo -e 'You have '$num$atom$end' new letter(s)'
`

