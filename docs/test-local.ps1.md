---
Author: andy arismendi
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6640
Published Date: 2016-11-30t00
Archived Date: 2016-11-30t18
---

# test.local - 

## Description

sets the system primary dns suffix by p-invoking the win32 api. returns true for success, false for failure.

## Comments



## Usage



## TODO



## function

`set-primarydnssuffix`

## Code

`#
 #
 function Set-PrimaryDnsSuffix {
 	param ([string] $Suffix)
 	
 	$ComputerNamePhysicalDnsDomain = 6
 	
 	Add-Type -TypeDefinition @"
 	using System;
 	using System.Runtime.InteropServices;
 
 	namespace ComputerSystem {
 	    public class renamefull {
 	        [DllImport("kernel32.dll", CharSet = CharSet.Auto)]
 	        static extern bool SetComputerNameEx(int NameType, string lpBuffer);
 	        public static bool SetPrimaryDnsSuffix(string suffix) {
 	            try {
 	                const int ComputerNamePhysicalDnsDomain = 6;
 	                return SetComputerNameEx(ComputerNamePhysicalDnsDomain, suffix);
 	            }
 	            catch (Exception) {
 	                return false;
 	            }
 	        }
 	    }
 	}
 "@
 	[ComputerSystem.Identification]::SetPrimaryDnsSuffix($Suffix)
 }
`

