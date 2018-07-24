---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4562
Published Date: 2013-10-26t16
Archived Date: 2013-11-01t01
---

# hex2dec - 

## Description

old bash script…

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 #!/bin/bash
 
 err="\033[1;31m"
 res="\033[1;36m"
 end="\033[0m"
 
    n=$1
    case $n in
       *[[:xdigit:]])
          if [[ ${n:0:2} =~ 0[xX] ]]; then
             echo -e $res$n' = '$(($n))$end
          elif [[ ${n:0:1} =~ [xX] ]]; then
             echo -e $res'0'$n' = '$((0$n))$end
          elif [[ ${n:0:1} =~ [[:digit:]] && ! $n =~ [[:alpha:]] ]]; then
             printf $res$n' = 0x%X\n'$end $n
          elif [[ ${n:0:1} =~ [[:xdigit:]] ]]; then
             echo -e $res'0x'$n' = '$((0x$n))$end
          else
             echo -e $err'=>err'$end
          fi;;
       *)
          echo -e $err'=>err'$end;;
    esac
 else
    echo -e $err'=>err'$end
 fi
`

