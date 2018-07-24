---
Author: hotsnoj
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3053
Published Date: 2011-11-16t09
Archived Date: 2011-11-22t10
---

# down with sopa! - 

## Description

let’s fill the logs of the us house and senate servers with the message we don’t want sopa or e-parasite!

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 .Synopsis
 Let's fill the logs of the US House and Senate servers with the message we don't want SOPA or E-Parasite!
 .Description
 Runs an while(1) loop that grabs a couple URI's from each branch's website and sleeps for 1 second between requests.
 #>
 
 
 while(1){
     try{
         (new-object net.webclient).downloadstring("http://www.house.gov/downWithSOPA") | Out-Null;
     } catch{}
     
     try{
         (new-object net.webclient).downloadstring("http://www.senate.gov/downWithE-Parasite") | out-n
     } catch{}
     
     sleep 1;
 }
`

