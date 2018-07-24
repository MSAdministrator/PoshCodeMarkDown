---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5270
Published Date: 2014-06-30t15
Archived Date: 2014-07-22t15
---

# live session fun - 

## Description

somebody says that there is no compatui (com object) in windows 7, someone says the opposite. i found it on my copy of win7 but i’m not sure what it’s a standard library – probably it’s a rudiment of winxp.

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 PS D:\> $dll = (gcm -ea 0 -c Application compatui.dll).Definition
 PS D:\> PS D:\> if ($dll -ne $null) {
 >> strings -n 7 $dll | ? {$_ -cmatch 'CompatUI|CLSID'}
 >> }
 >>
 ...
 CompatUI.Util = s 'Util Class'
 CLSID = s '{0355854A-7F23-47E2-B7C3-97EE8DD42CD8}'
 ...
 PS D:\> $cu = [Activator]::CreateInstance(([Type]::GetTypeFromCLSID('{0355854A-7F23-47E2B7C3-97EE8DD42CD8}', $false)))
 PS D:\> $cu -eq $null
 False
 PS D:\> $cu | gm
 
 
    TypeName: System.__ComObject#{c5a7c759-1e8d-4b1b-b1e4-59f7f9b5171b}
 
 Name                   MemberType Definition
 ----                   ---------- ----------
 CheckAdminPrivileges   Method     int CheckAdminPrivileges ()
 GetExePathFromObject   Method     Variant GetExePathFromObject (string)
 GetItemKeys            Method     Variant GetItemKeys (string)
 IsCompatWizardDisabled Method     int IsCompatWizardDisabled ()
 IsExecutableFile       Method     int IsExecutableFile (string)
 IsSystemTarget         Method     int IsSystemTarget (string)
 RemoveArgs             Method     Variant RemoveArgs (string)
 RunApplication         Method     uint RunApplication (string, string, int)
 SetItemKeys            Method     int SetItemKeys (string, Variant)
 
 
 PS D:\> [Boolean]$cu.CheckAdminPrivileges()
 False
 PS D:\> [Boolean]$cu.IsExecutableFile('D:\bin\sed.exe')
 True
 PS D:\> Out-File src\dummy -Encoding ASCII -InputObject 'MZ'
 PS D:\> [Boolean]$cu.IsExecutableFile('D:\src\dummy')
 True
 PS D:\>
`

