---
Author: boe prox
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2021
Published Date: 2010-07-25t16
Archived Date: 2016-11-07t08
---

# take-screenshot - 

## Description

this script has a function that allows you to take a screenshot of the entire desktop or of an active window.  also includes option to save the screenshot to a file.

## Comments



## Usage



## TODO



## function

`take-screenshot`

## Code

`#
 #
 Function Take-ScreenShot {
 .SYNOPSIS  
     Used to take a screenshot of the desktop or the active window. 
 .DESCRIPTION  
     Used to take a screenshot of the desktop or the active window and save to an image file if needed.
 .PARAMETER screen
     Screenshot of the entire screen
 .PARAMETER activewindow
     Screenshot of the active window
 .PARAMETER file
     Name of the file to save as. Default is image.bmp
 .PARAMETER imagetype
     Type of image being saved. Can use JPEG,BMP,PNG. Default is bitmap(bmp)    
 .INPUTS
 .OUTPUTS    
 .NOTES  
     Name: Take-ScreenShot
     Author: Boe Prox
     DateCreated: 07/25/2010     
 .EXAMPLE  
     Take-ScreenShot -activewindow
     Takes a screen shot of the active window        
 .EXAMPLE  
     Take-ScreenShot -Screen
     Takes a screenshot of the entire desktop
 .EXAMPLE  
     Take-ScreenShot -activewindow -file "C:\image.bmp" -imagetype bmp
     Takes a screenshot of the active window and saves the file named image.bmp with the image being bitmap
 .EXAMPLE  
     Take-ScreenShot -screen -file "C:\image.png" -imagetype png    
     Takes a screenshot of the entire desktop and saves the file named image.png with the image being png
 #>  
         [cmdletbinding(
                 SupportsShouldProcess = $True,
                 DefaultParameterSetName = "screen",
                 ConfirmImpact = "low"
         )]
 Param (
        [Parameter(
             Mandatory = $False,
             ParameterSetName = "screen",
             ValueFromPipeline = $True)]
             [switch]$screen,
        [Parameter(
             Mandatory = $False,
             ParameterSetName = "window",
             ValueFromPipeline = $False)]
             [switch]$activewindow,
        [Parameter(
             Mandatory = $False,
             ParameterSetName = "",
             ValueFromPipeline = $False)]
             [string]$file, 
        [Parameter(
             Mandatory = $False,
             ParameterSetName = "",
             ValueFromPipeline = $False)]
             [string]
             [ValidateSet("bmp","jpeg","png")]
             $imagetype = "bmp"           
        
 )
 $code = @'
 using System;
 using System.Runtime.InteropServices;
 using System.Drawing;
 using System.Drawing.Imaging;
 namespace ScreenShotDemo
 {
   /// <summary>
   /// Provides functions to capture the entire screen, or a particular window, and save it to a file.
   /// </summary>
   public class ScreenCapture
   {
     /// <summary>
     /// Creates an Image object containing a screen shot the active window
     /// </summary>
     /// <returns></returns>
     public Image CaptureActiveWindow()
     {
       return CaptureWindow( User32.GetForegroundWindow() );
     }
     /// <summary>
     /// Creates an Image object containing a screen shot of the entire desktop
     /// </summary>
     /// <returns></returns>
     public Image CaptureScreen()
     {
       return CaptureWindow( User32.GetDesktopWindow() );
     }    
     /// <summary>
     /// Creates an Image object containing a screen shot of a specific window
     /// </summary>
     /// <param name="handle">The handle to the window. (In windows forms, this is obtained by the Handle property)</param>
     /// <returns></returns>
     private Image CaptureWindow(IntPtr handle)
     {
       // get te hDC of the target window
       IntPtr hdcSrc = User32.GetWindowDC(handle);
       // get the size
       User32.RECT windowRect = new User32.RECT();
       User32.GetWindowRect(handle,ref windowRect);
       int width = windowRect.right - windowRect.left;
       int height = windowRect.bottom - windowRect.top;
       // create a device context we can copy to
       IntPtr hdcDest = GDI32.CreateCompatibleDC(hdcSrc);
       // create a bitmap we can copy it to,
       // using GetDeviceCaps to get the width/height
       IntPtr hBitmap = GDI32.CreateCompatibleBitmap(hdcSrc,width,height);
       // select the bitmap object
       IntPtr hOld = GDI32.SelectObject(hdcDest,hBitmap);
       // bitblt over
       GDI32.BitBlt(hdcDest,0,0,width,height,hdcSrc,0,0,GDI32.SRCCOPY);
       // restore selection
       GDI32.SelectObject(hdcDest,hOld);
       // clean up
       GDI32.DeleteDC(hdcDest);
       User32.ReleaseDC(handle,hdcSrc);
       // get a .NET image object for it
       Image img = Image.FromHbitmap(hBitmap);
       // free up the Bitmap object
       GDI32.DeleteObject(hBitmap);
       return img;
     }
     /// <summary>
     /// Captures a screen shot of the active window, and saves it to a file
     /// </summary>
     /// <param name="filename"></param>
     /// <param name="format"></param>
     public void CaptureActiveWindowToFile(string filename, ImageFormat format)
     {
       Image img = CaptureActiveWindow();
       img.Save(filename,format);
     }
     /// <summary>
     /// Captures a screen shot of the entire desktop, and saves it to a file
     /// </summary>
     /// <param name="filename"></param>
     /// <param name="format"></param>
     public void CaptureScreenToFile(string filename, ImageFormat format)
     {
       Image img = CaptureScreen();
       img.Save(filename,format);
     }    
    
     /// <summary>
     /// Helper class containing Gdi32 API functions
     /// </summary>
     private class GDI32
     {
       
       public const int SRCCOPY = 0x00CC0020; // BitBlt dwRop parameter
       [DllImport("gdi32.dll")]
       public static extern bool BitBlt(IntPtr hObject,int nXDest,int nYDest,
         int nWidth,int nHeight,IntPtr hObjectSource,
         int nXSrc,int nYSrc,int dwRop);
       [DllImport("gdi32.dll")]
       public static extern IntPtr CreateCompatibleBitmap(IntPtr hDC,int nWidth,
         int nHeight);
       [DllImport("gdi32.dll")]
       public static extern IntPtr CreateCompatibleDC(IntPtr hDC);
       [DllImport("gdi32.dll")]
       public static extern bool DeleteDC(IntPtr hDC);
       [DllImport("gdi32.dll")]
       public static extern bool DeleteObject(IntPtr hObject);
       [DllImport("gdi32.dll")]
       public static extern IntPtr SelectObject(IntPtr hDC,IntPtr hObject);
     }
 
     /// <summary>
     /// Helper class containing User32 API functions
     /// </summary>
     private class User32
     {
       [StructLayout(LayoutKind.Sequential)]
       public struct RECT
       {
         public int left;
         public int top;
         public int right;
         public int bottom;
       }
       [DllImport("user32.dll")]
       public static extern IntPtr GetDesktopWindow();
       [DllImport("user32.dll")]
       public static extern IntPtr GetWindowDC(IntPtr hWnd);
       [DllImport("user32.dll")]
       public static extern IntPtr ReleaseDC(IntPtr hWnd,IntPtr hDC);
       [DllImport("user32.dll")]
       public static extern IntPtr GetWindowRect(IntPtr hWnd,ref RECT rect);
       [DllImport("user32.dll")]
       public static extern IntPtr GetForegroundWindow();      
     }
   }
 }
 '@
 add-type $code -ReferencedAssemblies 'System.Windows.Forms','System.Drawing'
 $capture = New-Object ScreenShotDemo.ScreenCapture
 
 If ($Screen) {
     Write-Verbose "Taking screenshot of entire desktop"
     If ($file) {
         If ($file -eq "") {
             $file = "$pwd\image.bmp"
             }
         Write-Verbose "Creating screen file: $file with imagetype of $imagetype"
         $capture.CaptureScreenToFile($file,$imagetype)
         }
     Else {
         $capture.CaptureScreen()
         }
     }
 If ($ActiveWindow) {
     Write-Verbose "Taking screenshot of the active window"
     If ($file) {
         If ($file -eq "") {
             $file = "$pwd\image.bmp"
             }
         Write-Verbose "Creating activewindow file: $file with imagetype of $imagetype"
         $capture.CaptureActiveWindowToFile($file,$imagetype)
         }
     Else {
         $capture.CaptureActiveWindow()
         }    
     }    
 }
`

