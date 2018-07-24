---
Author: nathan hartley
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 882
Published Date: 2009-02-19t15
Archived Date: 2017-04-04t05
---

# get-netview - 

## Description

a one liner that parses the output of net.exe’s view command. net.exe view displays a list of computers in your current domain by default, to display another domain change it to read net.exe view /domain <domainname>.

## Comments



## Usage



## TODO



## function

`get-netview`

## Code

`#
 #
 function Get-NetView {
 	switch -regex (NET.EXE VIEW) { "^\\\\(?<Name>\S+)\s+" {$matches.Name}}
 	}
`

