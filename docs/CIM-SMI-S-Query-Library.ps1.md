---
Author: parul jain
Publisher: 
Copyright: 
Email: 
Version: 0.2
Encoding: ascii
License: cc0
PoshCode ID: 3985
Published Date: 2014-02-22t18
Archived Date: 2014-04-08t03
---

# cim smi-s query library - 

## Description

the function queries any cim server, called cimom, that supports the cim-xml interface. it creates an xml message to encapsulate the cim query, converts the message to byte stream and then sends it using http post method. the response byte stream is converted back to xml message and name value paris are parsed out. smi-s is an instance of cim, and is thus also fully supported.

## Comments



## Usage



## TODO



## function

`invoke-cim_enumerateinstances`

## Code

`#
 #
 function Invoke-CIM_EnumerateInstances {
 <#
 .SYNOPSIS
     Enumerate Instances of a class on a CIMOM via CIM-XML interface
 
 .DESCRIPTION
     Primary use case of this function is to gather inventory and performance information from IT
     infrastructure assets. The inventory information feeds into capacity planning, troubleshooting,
     managing product life cycle, budgeting, vendor price negotiations and technology strategy in
     large enterprise environments. The output from this function would typically go into a datawarehouse
     front ended with a business intelligence platform such as COGNOS, QlikView, Business Objects, etc.
 
     The function queries any CIM server, called CIMOM, that supports the CIM-XML interface. It
     creates an XML message to encapsulate the CIM query, converts the message to byte stream and
     then sends it using HTTP POST method. The response byte stream is converted back to XML message
     and name value pairs are parsed out. SMI-S is an instance of CIM, and is thus also fully supported.
 
     Tested against SAN devices such as EMC Symmetrix VMAX Fibre Channel Storage Array and Cisco MDS
     Fibre Channel switch. It can be used to query VMWARE vSphere vCenter, IBM XIV, NetApp Filer, EMC
     VNX Storage Array, HP Insight Manager, Dell OpenManage, HDS: USP, USPV, VSP, AMS, etc.
 
 .NOTES
     Author: Parul Jain (paruljain@hotmail.com)
     Version: 0.2, Jan, 2013
     Requires: PowerShell v2 or better
 
 .EXAMPLE
     This works with EMC Symmetrix
       
 .PARAMETER class
     Mandatory. Information within CIM is classified into classes. The device documentation (or SNIA
     documntation in case of SMI-S) should list all the classes supported by the CIMOM. CIM_ComputerSystem
     class is available universally and is a good place to start testing.
 
 .PARAMETER CIMServer
     Mandatory. IP address or DNS name of the device or CIMOM server if CIMOM runs outside the device
 
 .PARAMETER user
     Mandatory. User ID authorized to perform queries. Most hardware vendors have a factory default
 
 .PARAMETER pass
     Mandatory. Password for the user. Again most hardware vendors have a factory default for servicing the equipment
 
 .PARAMETER port
     Optional. The TCP port number that the CIMOM is listening to. Default is used if not specified.
 
 .PARAMETER ssl
     Optional switch. When used function will use HTTPS instead of default HTTP
 
 .PARAMETER ns
     Optional. CIM namespace to use. Default is root/cimv2. EMC uses root/emc
 
 .PARAMETER dbg
     Optional switch. Returns CIM-XML response message instead of parsed name-value pairs for
     troubleshooting parsing if needed
 #>
     
     [CmdletBinding()]
 
     Param (
         [parameter(Mandatory=$true, ValueFromPipeline=$true)][String]$Class,
         [parameter(Mandatory=$true, ValueFromPipeline=$false)][String]$User,
         [parameter(Mandatory=$true, ValueFromPipeline=$false)][String]$Pass,
         [parameter(Mandatory=$true, ValueFromPipeline=$false)][String]$CIMServer,
         [parameter(Mandatory=$false, ValueFromPipeline=$false)][String]$Port,
         [parameter(Mandatory=$false, ValueFromPipeline=$false)][Switch]$ssl,
         [parameter(Mandatory=$false, ValueFromPipeline=$false)][String]$ns = 'root/cimv2',
         [parameter(Mandatory=$false, ValueFromPipeline=$false)][Switch]$dbg
     ) 
 
 $messageText = @'
 <?xml version="1.0" encoding="utf-8" ?>
 
 <CIM CIMVERSION="2.0" DTDVERSION="2.0">
 
     <MESSAGE ID="1000" PROTOCOLVERSION="1.0">
         <SIMPLEREQ>
             <IMETHODCALL NAME="EnumerateInstances">
                 <LOCALNAMESPACEPATH>
                 </LOCALNAMESPACEPATH>
                 <IPARAMVALUE NAME="ClassName">
                     <CLASSNAME NAME="CIM_ComputerSystem" />
                 </IPARAMVALUE>
                 <IPARAMVALUE NAME="DeepInheritance">
                     <VALUE>FALSE</VALUE>
                 </IPARAMVALUE>
                 <IPARAMVALUE NAME="LocalOnly">
                     <VALUE>TRUE</VALUE>
                 </IPARAMVALUE>
                 <IPARAMVALUE NAME="IncludeClassOrigin">
                     <VALUE>FALSE</VALUE>
                 </IPARAMVALUE>
                 <IPARAMVALUE NAME="IncludeQualifiers">
                     <VALUE>FALSE</VALUE>
                 </IPARAMVALUE>
 
             </IMETHODCALL>
         </SIMPLEREQ>
     </MESSAGE>
 </CIM>
 '@
 
     $message = [xml]($messageText)
     ($message.CIM.MESSAGE.SIMPLEREQ.IMETHODCALL.IPARAMVALUE | where { $_.Name -eq 'ClassName' }).Classname.Name = $Class
 
     $nsNode = $message.SelectSingleNode('/CIM/MESSAGE/SIMPLEREQ/IMETHODCALL/LOCALNAMESPACEPATH')
     foreach ($item in $ns.split('/')) {
         $nsNode.InnerXml += '<NAMESPACE NAME="' + $item + '" />'
     }
 
     [Net.ServicePointManager]::ServerCertificateValidationCallback = { $true }
 
     if ($ssl) { $protocol = 'https://' } else { $protocol = 'http://' }
     if (!$port) { if ($ssl) { $port = '5989' } else { $port = '5988' } }
     
     $url = $protocol + $CIMServer + ":" + $port + '/cimom'
 
     $req = New-Object Net.WebClient
 
     $req.Headers.Add('Content-Type', 'application/xml;charset="UTF-8"')
     $req.Headers.Add('CIMOperation', 'MethodCall')
     $req.Headers.Add('CIMMethod', 'EnumerateInstances')
     $req.Headers.Add('CIMObject', $ns)
     $req.Headers.Add('Authorization', 'Basic ' + [Convert]::ToBase64String([text.encoding]::UTF8.GetBytes($user + ':' + $pass)))
 
     $result = [xml]($req.UploadString($url, $message.OuterXml))
     
     if ($dbg) { return $result }
 
     foreach ($instance in @($result.CIM.MESSAGE.SIMPLERSP.IMETHODRESPONSE.IRETURNVALUE.'Value.NamedInstance')) {
         $object = New-Object PSObject
         foreach ($prop in @($instance.instance.property)) {
             if ($prop.value -ne $null -and $prop.value -ne '') {
                 $object | Add-Member -MemberType NoteProperty -Name $prop.Name -Value $prop.Value
             }
         }
     }
 }
 
 function Invoke-CIM_GetInstance {
  
     [CmdletBinding()]
 
     Param (
         [parameter(Mandatory=$true, ValueFromPipeline=$true)][String]$Class,
         [parameter(Mandatory=$true, ValueFromPipeline=$false)][String]$User,
         [parameter(Mandatory=$true, ValueFromPipeline=$false)][String]$Pass,
         [parameter(Mandatory=$true, ValueFromPipeline=$false)][String]$CIMServer,
         [parameter(Mandatory=$false, ValueFromPipeline=$false)][String]$Port,
         [parameter(Mandatory=$false, ValueFromPipeline=$false)][Switch]$ssl,
         [parameter(Mandatory=$false, ValueFromPipeline=$false)][String]$ns = 'root/cimv2',
         [parameter(Mandatory=$false, ValueFromPipeline=$false)][Switch]$dbg,
         [parameter(Mandatory=$true, ValueFromPipeline=$false)][String]$key
     ) 
 
 $messageText = @'
 <?xml version="1.0" encoding="utf-8" ?>
 
 <CIM CIMVERSION="2.0" DTDVERSION="2.0">
 
     <MESSAGE ID="1000" PROTOCOLVERSION="1.0">
         <SIMPLEREQ>
             <IMETHODCALL NAME="GetInstance">
                 <LOCALNAMESPACEPATH>
                 </LOCALNAMESPACEPATH>
 
                 <IPARAMVALUE NAME="InstanceName">
                     <INSTANCENAME CLASSNAME="Symm_StorageSystem">
                         <KEYBINDING NAME="CreationClassName">
                             <KEYVALUE VALUETYPE="string">Symm_StorageSystem</KEYVALUE>
                         </KEYBINDING>
                         <KEYBINDING NAME="Name">
                             <KEYVALUE VALUETYPE="string">SYMMETRIX+000192601380</KEYVALUE>
                         </KEYBINDING>
                     </INSTANCENAME>
                 </IPARAMVALUE>
                 
                 <IPARAMVALUE NAME="LocalOnly">
                     <VALUE>FALSE</VALUE>
                 </IPARAMVALUE>
 
             </IMETHODCALL>
         </SIMPLEREQ>
     </MESSAGE>
 </CIM>
 '@
 
     $message = [xml]($messageText)
     ($message.CIM.MESSAGE.SIMPLEREQ.IMETHODCALL.IPARAMVALUE | where { $_.Name -eq 'InstanceName' }).InstanceName.ClassName = $Class
     (($message.CIM.MESSAGE.SIMPLEREQ.IMETHODCALL.IPARAMVALUE |
         where { $_.Name -eq 'InstanceName' }).instancename.keybinding | where { $_.Name -eq 'CreationClassName' }).KeyValue.InnerText = $Class
     (($message.CIM.MESSAGE.SIMPLEREQ.IMETHODCALL.IPARAMVALUE |
         where { $_.Name -eq 'InstanceName' }).instancename.keybinding | where { $_.Name -eq 'Name' }).KeyValue.InnerText = $Key
 
     $nsNode = $message.SelectSingleNode('/CIM/MESSAGE/SIMPLEREQ/IMETHODCALL/LOCALNAMESPACEPATH')
     foreach ($item in $ns.split('/')) {
         $nsNode.InnerXml += '<NAMESPACE NAME="' + $item + '" />'
     }
 
     [Net.ServicePointManager]::ServerCertificateValidationCallback = { $true }
 
     if ($ssl) { $protocol = 'https://' } else { $protocol = 'http://' }
     if (!$port) { if ($ssl) { $port = '5989' } else { $port = '5988' } }
     
     $url = $protocol + $CIMServer + ":" + $port + '/cimom'
 
     $req = New-Object Net.WebClient
 
     $req.Headers.Add('Content-Type', 'application/xml;charset="UTF-8"')
     $req.Headers.Add('CIMOperation', 'MethodCall')
     $req.Headers.Add('CIMMethod', 'GetInstance')
     $req.Headers.Add('CIMObject', $ns)
     $req.Headers.Add('Authorization', 'Basic ' + [Convert]::ToBase64String([text.encoding]::UTF8.GetBytes($user + ':' + $pass)))
 
     $result = [xml]($req.UploadString($url, $message.OuterXml))
     
     if ($dbg) { return $result }
 
     $object = New-Object PSObject
     foreach ($prop in @($result.CIM.MESSAGE.SIMPLERSP.IMETHODRESPONSE.IRETURNVALUE.INSTANCE.PROPERTY)) {
         if ($prop.value -ne $null -and $prop.value -ne '') {
             $object | Add-Member -MemberType NoteProperty -Name $prop.Name -Value $prop.Value
         }
     }
 }
 
 
 function Get-EMCStoragePools ($SMIserver) {
 }
 
 function Get-SNIADiskDriveView ($SMIServer) {
 }
 
 function Get-SymmStorageSystem ([string]$SMIServer, [string]$sid) {
     if (!$sid) {
     } else {
     }
 }
 
 function Get-SymmStorageVolume ($SMIServer) {
 }
`

