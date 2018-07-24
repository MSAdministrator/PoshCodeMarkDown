---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1998
Published Date: 
Archived Date: 2010-07-23t17
---

# comobjects.types - 

## Description

this is a .ps1xml file for use with update-typedata … it adds methods onto the com object for getproperty (including support for parametrized properties), setproperty, and invokemethod.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <Types>
    <Type>
       <Name>System.__ComObject</Name>
       <Members>
          <ScriptMethod>
             <Name>GetProperty</Name>
             <Script>
                param([Parameter(Mandatory=$true,Position=1)]$PropertyName)
                Write-Verbose "PropertyName: $PropertyName"
                Write-Verbose "Arguments: $($Args | Out-String)"
                $this.gettype().invokeMember($PropertyName,[System.Reflection.BindingFlags]::GetProperty,$null,$this,@($Args))
             </Script>
          </ScriptMethod>
          <ScriptMethod>
             <Name>SetProperty</Name>
             <Script>
                param([Parameter(Mandatory=$true,Position=1)]$PropertyName)
                $this.gettype().invokeMember($PropertyName,[System.Reflection.BindingFlags]::SetProperty,$null,$this,@($Args))
             </Script>
          </ScriptMethod>
          <ScriptMethod>
             <Name>InvokeMethod</Name>
             <Script>
                param([Parameter(Mandatory=$true,Position=1)]$MethodName)
                $this.gettype().invokeMember($MethodName,[System.Reflection.BindingFlags]::InvokeMethod,$null,$this,@($Args))
             </Script>
          </ScriptMethod>
       </Members>
    </Type>
 </Types>
`

