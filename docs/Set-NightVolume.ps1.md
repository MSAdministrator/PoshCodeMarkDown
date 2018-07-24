---
Author: jpruskin
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5991
Published Date: 2016-08-26t10
Archived Date: 2016-05-17t13
---

# set-nightvolume - 

## Description

sets device volume to 10% if run between 22

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 	.Synopsis
 	Sets volume to 10% if run between 10pm and 7am.
     .Notes
     Thanks to Alexandre Jasmin for the core audio API info.
     .Link
     .Link
     https://www.reddit.com/r/PowerShell/comments/3iea92/never_done_a_script_before_on_powershell_but_i/
 #>
 
 Add-Type -TypeDefinition @'
 
 using System.Runtime.InteropServices;
 [Guid("5CDF2C82-841E-4546-9722-0CF74078229A"), InterfaceType(ComInterfaceType.InterfaceIsIUnknown)]
 interface IAudioEndpointVolume {
   // f(), g(), ... are unused COM method slots. Define these if you care
   int f(); int g(); int h(); int i();
   int SetMasterVolumeLevelScalar(float fLevel, System.Guid pguidEventContext);
   int j();
   int GetMasterVolumeLevelScalar(out float pfLevel);
   int k(); int l(); int m(); int n();
   int SetMute([MarshalAs(UnmanagedType.Bool)] bool bMute, System.Guid pguidEventContext);
   int GetMute(out bool pbMute);
 }
 [Guid("D666063F-1587-4E43-81F1-B948E807363F"), InterfaceType(ComInterfaceType.InterfaceIsIUnknown)]
 interface IMMDevice {
   int Activate(ref System.Guid id, int clsCtx, int activationParams, out IAudioEndpointVolume aev);
 }
 [Guid("A95664D2-9614-4F35-A746-DE8DB63617E6"), InterfaceType(ComInterfaceType.InterfaceIsIUnknown)]
 interface IMMDeviceEnumerator {
   int f(); // Unused
   int GetDefaultAudioEndpoint(int dataFlow, int role, out IMMDevice endpoint);
 }
 [ComImport, Guid("BCDE0395-E52F-467C-8E3D-C4579291692E")] class MMDeviceEnumeratorComObject { }
 public class Audio {
   static IAudioEndpointVolume Vol() {
     var enumerator = new MMDeviceEnumeratorComObject() as IMMDeviceEnumerator;
     IMMDevice dev = null;
     Marshal.ThrowExceptionForHR(enumerator.GetDefaultAudioEndpoint(/*eRender*/ 0, /*eMultimedia*/ 1, out dev));
     IAudioEndpointVolume epv = null;
     var epvid = typeof(IAudioEndpointVolume).GUID;
     Marshal.ThrowExceptionForHR(dev.Activate(ref epvid, /*CLSCTX_ALL*/ 23, 0, out epv));
     return epv;
   }
   public static float Volume {
     get {float v = -1; Marshal.ThrowExceptionForHR(Vol().GetMasterVolumeLevelScalar(out v)); return v;}
     set {Marshal.ThrowExceptionForHR(Vol().SetMasterVolumeLevelScalar(value, System.Guid.Empty));}
   }
   public static bool Mute {
     get { bool mute; Marshal.ThrowExceptionForHR(Vol().GetMute(out mute)); return mute; }
     set { Marshal.ThrowExceptionForHR(Vol().SetMute(value, System.Guid.Empty)); }
   }
 }
 
 '@
 
 if ((Get-Date).Hour -in (22..23)+(0..6)) {[audio]::Volume  = 0.1}
`

