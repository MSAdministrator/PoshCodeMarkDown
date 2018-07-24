---
Author: karl prosser
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 864
Published Date: 2009-02-12t00
Archived Date: 2012-04-24t13
---

# foreach-withexception - 

## Description

simple function like foreach, but that traps exceptions using v2, and logs then in the $lastex variable. this way the pipeline can continue and process the objects that aren’t having errors happen against them. eventually this needs to be made much better, and be a v2 advanced function

## Comments



## Usage



## TODO



## function

`foreach-withexception`

## Code

`#
 #
 function foreach-withexception ([scriptblock]$process,$outputexception)
 {
   begin { $global:foreachex = @() }
   process { 
             try 
             {
             $local:inputitem = $_
             &$process 
             }
             catch
             {
             $local:exceptionitem = 1 | select-object object,exception
             $local:exceptionitem.object = $local:inputitem 
             $local:exceptionitem.exception = $_
             $global:foreachex += $local:exceptionitem
             }
           }
   end {}
 }
 set-alias %ex foreach-withexception
 
 100..-5 | %ex {  "yo $_" ;  1 / $_ }
 $foreachex
`

