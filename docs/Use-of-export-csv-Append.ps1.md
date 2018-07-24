---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3671
Published Date: 
Archived Date: 2012-10-21t16
---

#  - 

## Description

use of export-csv -append

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 Hi,
 
 I've tried your excellent -Append feature for export-csv and it works good. I've only have a short question. 
 
 Is it possible to make the append to new columns instead of rows? I'd like to combine two csv files as a part of an automation script and then I must get this to work.
 
 
 Import-csv  C:\Visiolizer\CI.csv | Select-Object * | Export-Csv C:\Visiolizer\Relation.csv -Append
 
 The above work with your source code imported to my $profile, but I'd like to append it to columns. Do you know of any solution?
 
 Would be very thankful for a response
 
 Regards
 Erik
`

