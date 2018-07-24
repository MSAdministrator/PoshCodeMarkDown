---
Author: mark e
Publisher: 
Copyright: 
Email: 
Version: 1.1
Encoding: ascii
License: cc0
PoshCode ID: 1588
Published Date: 2012-01-17t05
Archived Date: 2012-12-29t04
---

# get-apod - 

## Description

get-apod parses the astronomy picture of the day website and downloads the current day’s image. it then sets the image as the desktop wallpaper for the system.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 .SYNOPSIS
 	Gets the Astronomy Picture of the Day and sets it as your wallpaper.
 .DESCRIPTION
  	Get-Apod parses the Astronomy Picture of the Day website and downloads the current day's image. It then sets the image as the desktop wallpaper for the system.
 .LINK
 	http://antwrp.gsfc.nasa.gov/apod/astropix.html
 .EXAMPLE
 	C:PS> Get-Apod.ps1
 	
 	This example downloads the image and sets it as the desktop wallpaper.
 .EXAMPLE
 	c:PS> Get-Apod.ps1 C:\Images
 	
 	This example also downloads the image and sets it as the desktop wallpaper, but it allows you to choose the folder to download the pictures to. 
 .PARAMETER Folder
 	The folder were you want to download the pictures to.
 .NOTES
   Name:         Get-APOD.ps1
   Author:       Mark E. Schill
   Date Created: 12/24/2008
   Date Revised: 01/17/2010
   Version:      1.1
   History:      1.1 01/17/2010 - Updated Help information
   				1.0 12/24/2008 - Initial Revision
   
   ** Licensed under a Creative Commons Attribution 3.0 License ** 
 #>
 [CmdletBinding(SupportsShouldProcess=$False)]
 param
 (
 [Parameter(Position=0, Mandatory=$false, ValueFromPipeline=$false)]
 [String]$Folder = ((Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders" -Name "My Pictures")."My Pictures" )
 )
 
 add-type @"
 using System;
 using System.Runtime.InteropServices;
 using Microsoft.Win32;
 namespace Wallpaper
 {
    public enum Style : int
    {
        Tiled, Centered, Stretched, Fit
    }
 
 
    public class Setter {
       public const int SetDesktopWallpaper = 20;
       public const int UpdateIniFile = 0x01;
       public const int SendWinIniChange = 0x02;
 
       [DllImport("user32.dll", SetLastError = true, CharSet = CharSet.Auto)]
       private static extern int SystemParametersInfo (int uAction, int uParam, string lpvParam, int fuWinIni);
       
       public static void SetWallpaper ( string path, Wallpaper.Style style ) {
          SystemParametersInfo( SetDesktopWallpaper, 0, path, UpdateIniFile | SendWinIniChange );
          
          RegistryKey key = Registry.CurrentUser.OpenSubKey("Control Panel\\Desktop", true);
          switch( style )
          {
              case Style.Stretched :
                  key.SetValue(@"WallpaperStyle", "2") ; 
                  key.SetValue(@"TileWallpaper", "0") ;
                  break;
              case Style.Centered :
                  key.SetValue(@"WallpaperStyle", "1") ; 
                  key.SetValue(@"TileWallpaper", "0") ; 
                  break;
              case Style.Tiled :
                  key.SetValue(@"WallpaperStyle", "1") ; 
                  key.SetValue(@"TileWallpaper", "1") ;
                  break;
              case Style.Fit :
                  key.SetValue(@"WallpaperStyle", "6") ; 
                  key.SetValue(@"TileWallpaper", "0") ;
                  break;
          }
          key.Close();
       }
    }
 }
 "@ 
 
 if ( ! (Test-Path -path "$Folder\APODImages")) { mkdir $Folder\APODImages | Out-Null }
 $Web = New-Object System.Net.WebClient
 $Page = $Web.DownloadString("http://antwrp.gsfc.nasa.gov/apod/astropix.html")
 $Text = $Page.Replace("`n","")
 $RegEx = [regex]'<a href="image/(?<URL>.*?)">'
 
 $Text -match $RegEx | Out-Null
 $URL = $Matches['URL']
 
 $FileName = $Folder + "\APODImages\" + ($URL -split "/" | Select-Object -Last 1)
 $Address = "http://antwrp.gsfc.nasa.gov/image/" + $URL
 $Web.DownloadFile( $Address, $FileName )
 
 [Wallpaper.Setter]::SetWallpaper( (Convert-Path $FileName), "Fit" )
 
 $RegEx = [regex]'<b> Explanation: </b>(?<Text>.*?)<hr>'
 $Text -match $Regex | Out-Null
 $APODText = $Matches['Text']
 
`

