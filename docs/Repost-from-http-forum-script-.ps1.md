---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5786
Published Date: 
Archived Date: 2016-03-05t22
---

#  - 

## Description

repost from http

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 @echo off
   setlocal enabledelayedexpansion
     set "key=HKLM\SOFTWARE\Microsoft\.NETFramework"
     for /f "skip=3 tokens=3" %%i in (
       '2^>nul reg query %key% /v InstallRoot'
     ) do set "root=%%i"
     if "%root%" equ "" (
       echo Probably .NET Framework has not been installed.
       goto:eof
     )
     if not exist "%root%" (
       echo Specified path of .NET Framework is invalid.
       goto:eof
     )
     set "key=HKLM\SOFTWARE\Microsoft\Fusion\GACChangeNotification\Default"
     set /a "num=0"
     echo The Global Assembly Cache contains the following assemblies:
     for /f "skip=4" %%i in (
       'reg query %key% ^| findstr /v /irc:"StoreChangeID"'
     ) do (
       for /f "tokens=1,2,3,4,5 delims=," %%j in ("%%i") do (
         set "clt=%%l"
         if "!clt:~2,1!" neq "" (
           set "clt=neutral"
           set "tok=%%l"
           set "arc=%%m"
         ) else (
           set "tok=%%m"
           set "arc=%%n"
         )
         echo   %%j, Version=%%k, Culture=!clt!, PublicKeyToken=!tok!, ProcessorArchitecture=!arc!
       )
       set /a "num+=1"
     )
     echo.
     echo Number of items = !num!
   endlocal
 exit /b
`

