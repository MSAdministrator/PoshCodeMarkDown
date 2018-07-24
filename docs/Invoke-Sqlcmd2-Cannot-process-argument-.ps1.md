---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4138
Published Date: 
Archived Date: 2013-05-07t07
---

#  - 

## Description

invoke-sqlcmd2

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Invoke-Sqlcmd2 : Cannot process argument transformation on parameter 'ConnectionString'. Cannot convert the "System.Col
 lections.Hashtable" value of type "System.Collections.Hashtable" to type "System.Data.SqlClient.SqlConnectionStringBuil
 der".
 At C:\Users\1\Desktop\1.ps1:64 char:39
 +       Invoke-Sqlcmd2 -ConnectionString <<<<  $Connection -Query $Query
     + CategoryInfo          : InvalidData: (:) [Invoke-Sqlcmd2], ParameterBindin...mationException
     + FullyQualifiedErrorId : ParameterArgumentTransformationError,Invoke-Sqlcmd2
`

