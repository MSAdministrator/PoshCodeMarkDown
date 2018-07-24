---
Author: stan miller
Publisher: 
Copyright: 
Email: 
Version: 7.1
Encoding: ascii
License: cc0
PoshCode ID: 3759
Published Date: 2012-11-11t13
Archived Date: 2012-11-16t06
---

# get-pecoff - 

## Description

takes file-name and outputs fileobject with two more properties

## Comments



## Usage



## TODO



## function

`get-pecoff`

## Code

`#
 #
 function Get-Pecoff 
 {
       <#
   .SYNOPSIS
     Takes file-name and outputs fileobject with two more properties
     poff_characteristic and poff_machinetype
   .DESCRIPTION
   This function creates file objects with PE properties for characteristic and machinetype
   .EXAMPLE
   For output from get-childitem or any other cmdlet that produces file objects from Win7 x64 machine.
   Get-ChildItem "C:\Windows\System32"|where {$_.Extension -like ".dll" -or $_.Extension -like ".exe"}|Get-Pecoff|select poff_machinetype,poff_characteristic,fullname|format-table -autosize
     poff_machinetype poff_characteristic                                                                   FullName
     AMD64            EXECUTABLE,DLL,LARGE_ADDRESS_AWARE                                                    C:\Windows\System32\aaclient.dll
     AMD64            EXECUTABLE,DLL,LARGE_ADDRESS_AWARE                                                    C:\Windows\System32\accessibilitycpl.dll
     AMD64            EXECUTABLE,DLL,LARGE_ADDRESS_AWARE                                                    C:\Windows\System32\ACCTRES.dll
     AMD64            EXECUTABLE,DLL,LARGE_ADDRESS_AWARE                                                    C:\Windows\System32\acledit.dll
     AMD64            EXECUTABLE,DLL,LARGE_ADDRESS_AWARE                                                    C:\Windows\System32\aclui.dll
     AMD64            EXECUTABLE,DLL,LARGE_ADDRESS_AWARE                                                    C:\Windows\System32\acppage.dll
     AMD64            EXECUTABLE,DLL,LARGE_ADDRESS_AWARE                                                    C:\Windows\System32\acproxy.dll
     AMD64            EXECUTABLE,DLL,LARGE_ADDRESS_AWARE                                                    C:\Windows\System32\ActionCenter.dll
     AMD64            EXECUTABLE,DLL,LARGE_ADDRESS_AWARE                                                    C:\Windows\System32\ActionCenterCPL.dll
     AMD64            EXECUTABLE,DLL,LARGE_ADDRESS_AWARE                                                    C:\Windows\System32\ActionQueue.dll
     AMD64            EXECUTABLE,DLL,LARGE_ADDRESS_AWARE                                                    C:\Windows\System32\activeds.dll
     AMD64            EXECUTABLE,DLL,LARGE_ADDRESS_AWARE                                                    C:\Windows\System32\actxprxy.dll
     AMD64            EXECUTABLE,LARGE_ADDRESS_AWARE                                                        C:\Windows\System32\AdapterTroubleshooter.exe
                             o
                             o
     I386             EXECUTABLE,LINE_NUMS_STRIPPED,32BIT_MACHINE,LOCAL_SYMS_STRIPPED                       C:\Windows\System32\TsWpfWrp.exe
                             o
     AMD64            NET_RUN_FROM_SWAP,EXECUTABLE,REMOVABLE_RUN_FROM_SWAP,DLL,LARGE_ADDRESS_AWARE          C:\Windows\System32\xmllite.dll
     AMD64            EXECUTABLE,DLL,LARGE_ADDRESS_AWARE                                                    C:\Windows\System32\xmlprovi.dll
     AMD64            EXECUTABLE,DLL,LARGE_ADDRESS_AWARE                                                    C:\Windows\System32\xolehlp.dll
     AMD64            EXECUTABLE,DLL,LARGE_ADDRESS_AWARE                                                    C:\Windows\System32\XpsFilt.dll
     AMD64            EXECUTABLE,DLL,LARGE_ADDRESS_AWARE                                                    C:\Windows\System32\XpsGdiConverter.dll
     AMD64            EXECUTABLE,DLL,LARGE_ADDRESS_AWARE                                                    C:\Windows\System32\XpsPrint.dll
     AMD64            EXECUTABLE,DLL,LARGE_ADDRESS_AWARE                                                    C:\Windows\System32\XpsRasterService.dll
     AMD64            EXECUTABLE,LARGE_ADDRESS_AWARE                                                        C:\Windows\System32\xpsrchvw.exe
     AMD64            EXECUTABLE,DLL,LARGE_ADDRESS_AWARE                                                    C:\Windows\System32\xpsservices.dll
     AMD64            EXECUTABLE,DLL,LARGE_ADDRESS_AWARE                                                    C:\Windows\System32\XPSSHHDR.dll
     AMD64            EXECUTABLE,DLL,LARGE_ADDRESS_AWARE                                                    C:\Windows\System32\xpssvcs.dll
     AMD64            EXECUTABLE,LARGE_ADDRESS_AWARE                                                        C:\Windows\System32\xwizard.exe
     AMD64            EXECUTABLE,DLL,LARGE_ADDRESS_AWARE                                                    C:\Windows\System32\xwizards.dll
     AMD64            EXECUTABLE,DLL,LARGE_ADDRESS_AWARE                                                    C:\Windows\System32\xwreg.dll
     AMD64            EXECUTABLE,DLL,LARGE_ADDRESS_AWARE                                                    C:\Windows\System32\xwtpdui.dll
     AMD64            EXECUTABLE,DLL,LARGE_ADDRESS_AWARE                                                    C:\Windows\System32\xwtpw32.dll
     AMD64            EXECUTABLE,DLL,LARGE_ADDRESS_AWARE                                                    C:\Windows\System32\zgmprxy.dll
     AMD64            EXECUTABLE,DLL,LARGE_ADDRESS_AWARE                                                    C:\Windows\System32\zipfldr.dll
 Where AMD64 means 64 bit app.
 Where I386 means 32 bit app.
 There are other archetectures than AMD64, I384: AM33 AMD64 ARM ARMV7 EBC IA64 M32R MIPS16 MIPSFPU MIPSFPU16
                                    POWERPC POWERPCFP R4000 SH3 SH3DSP SH4 SH5 THUMB WCEMIPSV2
 
 Where NoPE means file did not use PE standard
     NoPE             NoPE                                                                                   C:\Windows\SysWOW64\typelib.dll
 Where:
 RELOCS_STRIPPED = 0x1,
     Relocation info stripped from file.
 EXECUTABLE = 0x2,
     File is executable  (i.e. no unresolved external references).
 LINE_NUMS_STRIPPED = 0x4,
     Line numbers stripped from file.
 LOCAL_SYMS_STRIPPED = 0x8,
     Local symbols stripped from file.
 AGGRESSIVE_WS_TRIM = 0x10,
     Aggressively trim working set
 LARGE_ADDRESS_AWARE = 0x20,
 	For x86 applications:
 		on x86 OS
 			With special effort can allow access to 3Gb
 		on x64 OS
 			application can access 4GB
 	For x64 applications
 		on x64 OS
 			application can access 64 bits.
 BYTES_REVERSED_LO = 0x80,
     Bytes of machine word are reversed.
 32BIT_MACHINE = 0x100,
     32 bit word machine.
 DEBUG_STRIPPED = 0x200,
     Debugging info stripped from file in .DBG file
 REMOVABLE_RUN_FROM_SWAP = 0x400,
     If Image is on removable media, copy and run from the swap file.
     Not Sure Why system32 files would have this but they are on my 64bit win 7
 NET_RUN_FROM_SWAP = 0x800,
     If Image is on Net, copy and run from the swap file.
     Not Sure Why system32 files would have this but they are on my 64bit win 7
 SYSTEM = 0x1000,
     System File.
 DLL = 0x2000,
     File is a DLL.
 UP_SYSTEM_ONLY = 0x4000,
     File should only be run on a UP machine
 BYTES_REVERSED_HI = 0x8000
     Bytes of machine word are reversed.
 
 If get-pecoff cannot determine the file characteristic it will display the hex instead.
 This has been useful in debugging the table.
 
   .EXAMPLE
   For a single file:
   Get-Pecoff -filename "C:\Program Files\Microsoft SDKs\Windows\v7.1\Bin\x64\MdbgCore.dll"|select poff_machinetype,poff_characteristic,fullname|format-table -autosize
   .PARAMETER filename
   The full path of file to query.
   Get-Pecoff can take the output of getchilditem
   Creates file object output that has pe characteristic and pe machinetype properties added
 
   #>
   [CmdletBinding()]
   param
   (
     [Parameter(Mandatory=$True,ValueFromPipeline=$True,ValueFromPipelinebyPropertyName=$True)]
     [Alias('fullname')]
     [string]$filename=""
   )
 process {
     #"filename " + $filename
     $characteristics = @{}
     $characteristics["RELOCS_STRIPPED"] = 0x0001
     $characteristics["EXECUTABLE"] = 0x0002
     $characteristics["LINE_NUMS_STRIPPED"] = 0x0004
     $characteristics["LOCAL_SYMS_STRIPPED"] = 0x0008
     $characteristics["AGGRESSIVE_WS_TRIM"] = 0x0010
     $characteristics["LARGE_ADDRESS_AWARE"] = 0x0020
     $characteristics["RESERVED"] = 0x0040
     $characteristics["BYTES_REVERSED_LO"] = 0x0080
     $characteristics["32BIT_MACHINE"] = 0x0100
     $characteristics["DEBUG_STRIPPED"] = 0x0200
     $characteristics["REMOVABLE_RUN_FROM_SWAP"] = 0x0400
     $characteristics["NET_RUN_FROM_SWAP"] = 0x0800
     $characteristics["SYSTEM"] = 0x1000
     $characteristics["DLL"] = 0x2000
     $characteristics["UP_SYSTEM_ONLY"] = 0x4000
     $characteristics["BYTES_REVERSED_HI"] = 0x8000
     #$fileBytes = Get-Content $filename -ReadCount 0 -Encoding byte
     $stream=[System.IO.File]::OpenRead($filename)
     $fileBytes = New-Object byte[] 1024
     $bytesRead = $stream.Read($fileBytes,0,1024)
     $signatureOffset = 256*$fileBytes[0x3d]+$fileBytes[0x3c]
     #$signatureoffset
     $signature = [char[]] $fileBytes[$signatureOffset..($signatureOffset + 3)]
     if([String]::Join('', $signature) -ieq "PE`0`0")
     {
         $coffHeader = $signatureOffset + 4
         $characteristicsData = [BitConverter]::ToInt32($fileBytes, $coffHeader + 18)
         $poff_characteristic=""
         foreach($key in $characteristics.Keys)
         {
             $flag = $characteristics[$key]
             if(($characteristicsData -band $flag) -eq $flag)
             {
                 if ($poff_characteristic -eq "") {$poff_characteristic=$key} else {$poff_characteristic=$poff_characteristic+","+$key}
             }
         }
         $machineDatabin=[BitConverter]::ToInt32($fileBytes, $coffHeader)
         $machineData="0x"+[STRING]::Format("{0:X}",$machineDatabin) #.substring(1)
         #$machineData
         $machineTypes=@{}
         $machineTypes["0x"] = "UNKNOWN"
         $machineTypes["0x1d3"] = "AM33"
         $machineTypes["0x8664"] = "AMD64"
         $machineTypes["0x1c0"] = "ARM"
         $machineTypes["0x1c4"] = "ARMV7"
         $machineTypes["0xebc"] = "EBC"
         $machineTypes["0x14c"] = "I386"
         $machineTypes["0x200"] = "IA64"
         $machineTypes["0x9041"] = "M32R"
         $machineTypes["0x266"] = "MIPS16"
         $machineTypes["0x366"] = "MIPSFPU"
         $machineTypes["0x466"] = "MIPSFPU16"
         $machineTypes["0x1f0"] = "POWERPC"
         $machineTypes["0x1f1"] = "POWERPCFP"
         $machineTypes["0x166"] = "R4000"
         $machineTypes["0x1a2"] = "SH3"
         $machineTypes["0x1a3"] = "SH3DSP"
         $machineTypes["0x1a6"] = "SH4"
         $machineTypes["0x1a8"] = "SH5"
         $machineTypes["0x1c2"] = "THUMB"
         $machineTypes["0x169"] = "WCEMIPSV2"
         $poff_machinetype=$machineTypes[$machineData]
         if ($poff_machinetype -eq "" -or $poff_machinetype -eq $null) {$poff_machinetype=$machineData}
     }
     else
     {
         $poff_characteristic="NoPE"
         $poff_machinetype="NoPE"
     }
     $obj=get-childitem $filename
     $obj|Add-Member -MemberType NoteProperty "poff_characteristic" $poff_characteristic
     $obj|Add-Member -MemberType NoteProperty "poff_machinetype" $poff_machinetype
     #$obj|gm
     Write-Output $obj
     }
 }
`

