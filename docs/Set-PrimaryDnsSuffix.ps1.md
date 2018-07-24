---
Author: natedog
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6350
Published Date: 2016-05-17t23
Archived Date: 2016-05-20t15
---

# set-primarydnssuffix - 

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
 	    public class Identification {
 	        [DllImport("kernel32.dll", CharSet = CharSet.Auto)]
 	        static extern bool SetComputerNameEx(int NameType, string lpBuffer);
 			
 	        public static bool SetPrimaryDnsSuffix(string suffix) {
 	            try {
 	                return SetComputerNameEx($ComputerNamePhysicalDnsDomain, suffix);
 	            }
 	            catch (Exception) {
 	                return false;
 	            }
 	        }
 	    }
 	}
 "@
 	[ComputerSystem.Identification]::SetPrimaryDnsSuffix($happy.com)
 }
`

