---
Author: morlokor
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5568
Published Date: 2014-11-04t10
Archived Date: 2014-11-23t11
---

# user lookup script - 

## Description

mass user lookup script to query user account name / uid and resolve a user’s first last name created by morne loubser. this simple light-weight script can resolve as many usernames as you’d like to in one go. there aren’t any sources that performs this function on the net, so this one is a definite keeper. the first of it’s kind to date to make it’s way onto the web. can be used by both noobies and professional scripters.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 ***********************************************************************
 * Developed by: Morne Loubser (aka Morlokor)
 * Purpose: To mass query Active Directory for user first, last and display names
 * Date: 11/04/2014
 ***********************************************************************
 $in_file = "C:\*LOCATION_TO_YOUR_FILE*\users.csv"
 $out_file = "C:\*LOCATION_TO_YOUR_FILE*\users_out.csv"
 
 $out_data = @()
 
 ForEach ($row in (Import-Csv $in_file)) {
     If ($row.'user') {
         $out_data += Get-ADUser $row.'user' -Properties displayname
     }
 } 
 
 $out_data | 
 Select SamAccountName,Mail,Name,Surname,DisplayName | 
 Export-Csv -Path $out_file -NoTypeInformation
`

