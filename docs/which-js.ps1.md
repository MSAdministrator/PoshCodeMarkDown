---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5117
Published Date: 2014-04-24t15
Archived Date: 2014-08-29t23
---

# which.js - 

## Description

looks for a file into path variable.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 (function() {
   var fso = new ActiveXObject('Scripting.FileSystemObject'),
       wsh = new ActiveXObject('WScript.Shell'),
       lst = function(e) {
         return wsh.ExpandEnvironmentStrings(e).split(';');
       },
       dir = lst('%PATH%'),
       ext = lst('%PATHEXT%;.DLL'),
       itm, i, j;
   
   try {
     for (i in dir) {
       for (j in ext) {
         itm = dir[i] + '\\' + WScript.Arguments.Unnamed(0) + ext[j].toLowerCase();
         if (fso.FileExists(itm)) WScript.echo(itm);
       }
     }
   }
   catch (e) { WScript.echo(e.name + ': ' + e.message + '.'); }
 }());
`

