---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5654
Published Date: 2015-12-27t19
Archived Date: 2015-01-04t10
---

# visual cmd\bat - 

## Description

save script with .cmd or .bat extension and launch. this demo just shows current directory entries.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <!-- :
 @echo off
   setlocal
     start "" mshta.exe "%~f0"
   endlocal
 exit /b
 -->
 
 <!DOCTYPE html>
 <html>
   <head>
     <title>Visual CMD\BAT</title>
     <style type="text/css">
       body { font-family: tahoma; margin: 0 5; padding: 0; }
       p { margin: 0; padding: 0; }
     </style>
     <script language="JScript">
       function resize() { window.window.resizeTo(300, 300); }
     </script>
   </head>
   <body onload="resize();">
     <script>
       (function() {
         var std, arr, i;
         with (new ActiveXObject('WScript.Shell')) {
           std = Exec('cmd /q /k echo off');
           std.StdIn.WriteLine('dir /b & exit');
           arr = std.StdOut.ReadAll().split('\n');
           
           for (i = 0; i < arr.length; i++) {
             document.write('<p>' + arr[i] + '</p>');
           } //for
         }
       }());
     </script>
   </body>
 </html>
`

