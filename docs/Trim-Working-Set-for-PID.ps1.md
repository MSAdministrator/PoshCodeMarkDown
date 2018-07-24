---
Author: sunnychakraborty
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3653
Published Date: 2013-09-19t22
Archived Date: 2013-07-19t04
---

# trim working set for pid - 

## Description

trim working set for a pid.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 
 Function TrimWorkingSet {
 param([int] $procid)
 <#.NOTES
 AUTHOR: Sunny Chakraborty(sunnyc7@gmail.com)
 WEBSITE: http://tekout.wordpress.com
 CREATED: 9/20/2012
 This starts the Evil Monkey series of scripts.
  
 .DESCRIPTION
 MSDN - http://msdn.microsoft.com/en-us/library/windows/desktop/ms686234(v=vs.85).aspx
 Trim's working set to minimum levels (-1, -1)
 You can give a max / min values by modifying the signature.
 
 .WARNING
 ***********READ CAREFULLY***********
 !!!! Do not use in production environment before thoroughly testing and understanding the script. !!!!
 
 Do not use this to Trim Working Set for SQL Databases, there will be data-loss.
 Do not use this to Trim Working Set database for msExchange store.exe
 I havent tried this on IIS w3wp.exe processes.
 More NEGATIVE effects here - http://support.microsoft.com/kb/2001745
 
 I have primarily used this to trim browser WorkingSet data. Tested on firefox / iexplore / chrome. It works with no tab crashing.
 Trimming Working Set data, doesnt mean that browser Working Set values wont climb to their previous numbers.
 I have seen some IE windows go back to similar working set numbers.
 
 However, this script is really useful for a stuck browser and freezing screeing situations.
 ***********READ CAREFULLY***********
 
 .EXAMPLE
 TrimWorkingSet(1920)
 #>
 
 $sig = @"
 [DllImport("kernel32.dll")]
 public static extern bool SetProcessWorkingSetSize( IntPtr proc, int min, int max );
 "@
 
 $apptotrim = (get-process -Id $procid).Handle
 Add-Type -MemberDefinition $sig -Namespace User32 -Name Util -UsingNamespace System.Text -PassThru
 [User32.Util]::SetProcessWorkingSetSize($apptotrim,-1,-1)
 }
 TrimWorkingSet(5960)
`

