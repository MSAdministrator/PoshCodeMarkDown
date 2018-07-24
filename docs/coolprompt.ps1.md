---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1788
Published Date: 
Archived Date: 2010-04-25t07
---

# coolprompt - 

## Description

a cool prompt function.  insert code into your profile script (“c

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 
 	$global:wmilocalcomputer = get-WMIObject -class Win32_OperatingSystem -computer "."
 	$global:lastboottime=[System.Management.ManagementDateTimeconverter]::ToDateTime($wmilocalcomputer.lastbootuptime)
 	$global:originaltitle = [console]::title
 
 function prompt 
 {
 	$up=$(get-date)-$lastboottime
 
 	$upstr="$([datetime]::now.toshorttimestring()) $([datetime]::now.toshortdatestring()) up $($up.days) days, $($up.hours) hours, $($up.minutes) minutes"
 
 	$dir = $pwd.path
 
 	$homedir = (get-psprovider 'FileSystem').home
 
 	if ($homedir -ne "" -and $dir.toupper().startswith($homedir.toupper()))
 	{
 		$dir=$dir.remove(0,$homedir.length).insert(0,'~')
 	}
 	
 
 
 }
`

