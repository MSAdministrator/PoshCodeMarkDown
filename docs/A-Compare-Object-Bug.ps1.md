---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1069
Published Date: 
Archived Date: 2009-05-04t20
---

# a compare-object bug - 

## Description

demo for unexpected output of compare-object

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
  
 1, 2, 3, 4, 5 > .\textfile_a.txt
 1, 2, 4, 5, 6 > .\textfile_b.txt
 cat .\textfile_a.txt
 cat .\textfile_b.txt
 compare-object (gc .\textfile_a.txt) (gc .\textfile_b.txt) -inc
 InputObject                                SideIndicator                             
 -----------                                -------------                             
 1                                          ==                                        
 2                                          ==                                        
 4                                          ==                                        
 5                                          ==                                        
 6                                          =>                                        
 3                                          <=                                        
 
 I think it has to be
 InputObject                                SideIndicator                             
 -----------                                -------------                             
 1                                          ==                                        
 2                                          ==                                        
 3                                          <=                                        
 4                                          ==                                        
 5                                          ==                                        
 6                                          =>                                        
 #>
`

