---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 4.0
Encoding: ascii
License: cc0
PoshCode ID: 2969
Published Date: 2012-09-26t10
Archived Date: 2017-05-22t04
---

# powershell ise config - 

## Description

enable legacyv2runtimeactivation so that bitstransfer and sqlps will work in the new .net 4 powershell ise (this config file is required for powershell 3 ctp1 to work with those modules and other down-level .net 2 modules). save as c

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 <configuration>
    <startup useLegacyV2RuntimeActivationPolicy="true">
       <supportedRuntime version="v4.0" />
    </startup>
    <runtime>
       <loadFromRemoteSources enabled="true"/>
    </runtime>
 </configuration>
`

