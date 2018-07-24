---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.5
Encoding: ascii
License: cc0
PoshCode ID: 6642
Published Date: 2016-11-30t02
Archived Date: 2016-11-30t18
---

# set-wallpaper (ctp2) - 

## Description

set-wallpaper lets you set your windows desktop wallpaper.  it requires pinvoke and i wrote it using ctp2’s add-type, although it could be done in v1 using the jit code generation tricks lee holmes has mentioned in the past …

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 ###################################################################################################
 ###################################################################################################
 ###################################################################################################
 
 add-type @"
 using System;
 using System.Runtime.InteropServices;
 using Microsoft.Win32;
 namespace Wallpaper
 {
    public enum Style : int
    {
        Tile, Center, Stretch, NoChange
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
             case Style.Stretch :
                key.SetValue(@"WallpaperStyle", "2") ; 
                key.SetValue(@"TileWallpaper", "0") ;
                break;
             case Style.Center :
                key.SetValue(@"WallpaperStyle", "1") ; 
                key.SetValue(@"TileWallpaper", "0") ; 
                break;
             case Style.Tile :
                key.SetValue(@"WallpaperStyle", "1") ; 
                key.SetValue(@"TileWallpaper", "1") ;
                break;
             case Style.NoChange :
                break;
          }
          key.Close();
       }
    }
 }
 "@
 
 cmdlet Set-Wallpaper {
 Param(
    [Parameter(Position=0, Mandatory=$true, ValueFromPipelineByPropertyName=$true)]
    [Alias("FullName")]
    [string]
    $Path
 ,
    [Parameter(Position=1, Mandatory=$false)]
    [Wallpaper.Style]
    $Style = "NoChange"
 )
    [Wallpaper.Setter]::SetWallpaper( (Convert-Path $Path), $Style )
 }
`

