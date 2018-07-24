---
Author: paulo morgado
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5081
Published Date: 2014-04-15t11
Archived Date: 2014-08-20t09
---

# build sql server conn st - 

## Description

build a sql server connection string by specifying its parameters.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 param (
     [System.Data.SqlClient.ApplicationIntent]$ApplicationIntent,
     [string]$ApplicationName,
     [switch]$AsynchronousProcessing,
     [string]$AttachDBFilename,
     [switch]$ConnectionReset,
     [string]$ConnectionString,
     [int]$ConnectRetryCount,
     [int]$ConnectRetryInterval,
     [int]$ConnectTimeout,
     [switch]$ContextConnection,
     [string]$CurrentLanguage,
     [string]$DataSource,
     [switch]$Encrypt,
     [switch]$Enlist,
     [string]$FailoverPartner,
     [string]$InitialCatalog,
     [switch]$IntegratedSecurity,
     [int]$LoadBalanceTimeout,
     [int]$MaxPoolSize,
     [int]$MinPoolSize,
     [switch]$MultipleActiveResultSets,
     [switch]$MultiSubnetFailover,
     [string]$NetworkLibrary,
     [int]$PacketSize,
     [string]$Password,
     [switch]$PersistSecurityInfo,
     [switch]$Pooling,
     [switch]$Replication,
     [string]$TransactionBinding,
     [switch]$TrustServerCertificate,
     [string]$TypeSystemVersion,
     [string]$UserID,
     [switch]$UserInstance,
     [string]$WorkstationID,
     [Switch]$AsBuilder
 )
 
 if ($PSBoundParameters.ContainsKey('AsBuilder')) {
     $PSBoundParameters.Remove('AsBuilder') | Out-Null
 }
 
 $Builder = New-Object -TypeName System.Data.SqlClient.SqlConnectionStringBuilder -Property $PSBoundParameters
 
 if($AsBuilder) {
     $Builder
 } else {
     $Builder.ConnectionString
 }
`

