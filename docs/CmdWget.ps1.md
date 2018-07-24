---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 1.01
Encoding: ascii
License: cc0
PoshCode ID: 4437
Published Date: 2016-09-03t11
Archived Date: 2016-05-31t08
---

# cmdwget - 

## Description

this is command line script that invoke cscript by itself to download a file. just save this code with cmd or bat extension and lanch, for example, with next way

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 @set @script=0 /*
   @echo off
     set @script=
     setlocal
       set "i=0"
       for %%i in (%*) do set /a "i+=1"
       if "%i%" equ "2" (
         cscript //nologo //e:jscript "%~dpnx0" %1 %2
       ) else (
         echo %~n0 v1.01 - download files from internet
         echo Copyright ^(C^) 2010-2013 greg zakharov gregzakh@gmail.com
         echo.
         echo Usage: %~n0 ^<local_path^> ^<url^>
         echo.
         echo Example:
         echo   %~n0 E:\arc http://download.sysinternals.com/files/Autoruns.zip
       )
     endlocal
   exit /b
 */
 
 with (WScript.Arguments) {
   with (new ActiveXObject('Scripting.FileSystemObject')) {
     var file = Unnamed(1).substring(Unnamed(1).lastIndexOf('/') + 1, Unnamed(1).length);
     
     if (FolderExists(Unnamed(0))) {
       with (new ActiveXObject('MSXML2.XMLHTTP.3.0')) {
         try {
           open('GET', Unnamed(1), false);
           send();
         }
         catch (e) {}
         
         if (status == 200) {
           with (new ActiveXObject('ADODB.Stream')) {
             Open();
             Type = 1;
             Write(responseBody);
             SaveToFile(Unnamed(0) + '\\' + file);
             Close();
           }
         }
         else WScript.echo('Error: bad request.');
       }
     }
     else WScript.echo('Error: folder should be exist.');
   }
 }
`

