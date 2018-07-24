---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2720
Published Date: 2014-06-09t10
Archived Date: 2014-02-28t08
---

# new-sqlconnectionstring - 

## Description

build a sql connection string using specified parameters

## Comments



## Usage



## TODO



## function

`new-sqlconnectionstring`

## Code

`#
 #
 function New-SqlConnectionString {
 #.Synopsis
 #.Description
 #.Example
 #.Example
 [CmdletBinding(DefaultParameterSetName='Default')]
 PARAM(
    [String]${ConnectionString},
    [String]${ApplicationName},
    [Switch]${AsynchronousProcessing},
    [String]${AttachDBFilename},
    [String]${ConnectTimeout},
    [Switch]${ContextConnection},
    [String]${CurrentLanguage},
    [Parameter(Position=0)]
    [Alias("Server","Address")]
    [String]${DataSource},
    [Switch]${Encrypt},
    [Switch]${Enlist},
    [String]${FailoverPartner},
    [Parameter(Position=1)]
    [Alias("Database")]
    [String]${InitialCatalog},
    [Switch]${IntegratedSecurity},
    [Int]${LoadBalanceTimeout},
    [Int]${MaxPoolSize},
    [Int]${MinPoolSize},
    [Switch]${MultipleActiveResultSets},
    [String]${NetworkLibrary},
    [Int]${PacketSize},
    [AllowEmptyString()]
    [String]${Password},
    [Switch]${PersistSecurityInfo},
    [Switch]${Pooling},
    [Switch]${Replication},
    [String]${TransactionBinding},
    [Switch]${TrustServerCertificate},
    [String]${TypeSystemVersion},
    [Alias("UserName","Login")]
    [String]${UserID},
    [Switch]${UserInstance},
    [String]${WorkstationID},
    [Switch]${AsBuilder}
 )
 BEGIN {
    if(!( 'System.Data.SqlClient.SqlConnectionStringBuilder' -as [Type] )) {
      $null = [Reflection.Assembly]::Load( 'System.Data, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089' ) 
    }
 }
 PROCESS {
    $Builder = New-Object System.Data.SqlClient.SqlConnectionStringBuilder -Property $PSBoundParameters
    if($AsBuilder) {
       Write-Output $Builder
    } else {
       Write-Output $Builder.ToString()
    }
 }
 }
`

