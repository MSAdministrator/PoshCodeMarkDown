---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 622
Published Date: 
Archived Date: 2008-10-05t17
---

# connect-domain - 

## Description

we have multiple domains.  i added this function to my profile for each of our domains to easily connect to them using quest activeroles management shell for active directory.  the function kicks you into a nested prompt to work in the domain that you connect to and lets you exit back out.  it also modifies your foregroundcolor so you remember that you’re in a nested prompt.

## Comments



## Usage



## TODO



## function

`connect-domain_x`

## Code

`#
 #
 function connect-domain_X {
 		BEGIN {$foregroundcolor= (get-host).ui.rawui.get_foregroundcolor()
 			Write-Host "";
 					"---------------------------------" ;
 					"Entering Nested Prompt for Quest connection to DOMAIN_X."; 
 					"Type `"Exit`" when finished.";
 					"---------------------------------" ;
 					""
 					
 			(get-host).ui.rawui.set_foregroundcolor("magenta")
 			$pw = Read-Host "Enter your DOMAIN_X password" -AsSecureString
 					}
 		PROCESS {connect-QADService -service 'domaincontroller' -ConnectionAccount 'domain_x\username' -ConnectionPassword $pw
 			$host.enternestedprompt()
 		}
 		END {
 			(get-host).ui.rawui.set_foregroundcolor($foregroundcolor)
 		}
 	}
`

