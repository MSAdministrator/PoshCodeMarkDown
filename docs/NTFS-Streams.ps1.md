---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 1458
Published Date: 
Archived Date: 2009-11-13t08
---

# ntfs streams - 

## Description

provides access to ntfs alternate data streams, including functions for blocking and unblocking downloaded files. tweet @jaykul if you find problems.

## Comments



## Usage



## TODO



## module

`set-downloadflag`

## Code

`#
 #
 
 $PSScriptRoot = Split-Path $MyInvocation.MyCommand.Path -Parent
 
 function Set-DownloadFlag {
 <#
 .Synopsis
 	Sets the ZoneTransfer flag which marks a file as being downloaded from the internet.
 .Description
 	Creates a Zone.Identifier alternate data stream (on NTFS file systems) and writes the ZoneTransfer marker
 .Parameter Path
 	The file you wish to block
 .Parameter Passthru
 	If set, outputs the FileInfo object
 .Parameter ZoneId
    THe Zone you want to mark the file with. Defaults to 4
 #>
 [CmdletBinding()]
 PARAM (
    [Parameter(Position=0, Mandatory=$true, ValueFromPipelineByPropertyName=$true)]
    [Alias("FullName")]
    [string]
    $Path
 ,
    [Parameter(Position=1, Mandatory=$false)]
    [ZoneIdentifier]$Zone = "Untrusted"
 ,
    [Switch]$Passthru
 )
 PROCESS {
 
    $FS = new-object NTFS.FileStreams($Path)
    $null = $fs.add('Zone.Identifier')
    $stream = $fs.Item('Zone.Identifier').open()
 
    $sw = [System.IO.streamwriter]$stream
    $Sw.writeline('[ZoneTransfer]')
    $sw.writeline("ZoneID=$([Int]$zone)")
    $sw.close()
    $stream.close()
    if($Passthru){ Get-ChildItem $Path }
 }
 }
 
 function Remove-DownloadFlag {
 <#
 .Synopsis
 	Removes the ZoneTransfer flag which marks a file as being downloaded from the internet.
 .Description
 	Deletes the Zone.Identifier alternate data stream (on NTFS file systems)
 .Parameter Path
 	The file you wish to block
 .Parameter Passthru
 	If set, outputs the FileInfo object
 #>
 [CmdletBinding()]
 PARAM (
    [Parameter(Position=0, Mandatory=$true, ValueFromPipelineByPropertyName=$true)]
    [Alias("FullName")]
    [string]
    $Path
 ,
    [Switch]$Passthru
 )
 PROCESS {
    Remove-Stream -Path $Path -Name 'Zone.Identifier'
    if($Passthru){ Get-ChildItem $Path }
 }
 }
 
 function Get-DownloadFlag {
 <#
 .Synopsis
 	Verify whether the ZoneTransfer flag is set (marking this file as one downloaded from the internet).
 .Description
 	Reads the Zone.Identifier alternate data stream (on NTFS file systems)
 .Parameter Path
 	The file you wish to check the ZoneTransfer flag on
 #>
 [CmdletBinding()]
 PARAM (
    [Parameter(Position=0, Mandatory=$true, ValueFromPipelineByPropertyName=$true)]
    [Alias("FullName")]
    [string]
    $Path
 )
 Process { 
    $FS = new-object NTFS.FileStreams($Path)
    if(!$fs.Item('Zone.Identifier') ) {
       Write-Warning "Zone.Identifier not set on $Path (no Download Flag). This is the equivalent of a 'Trusted' flag."
       return
    }
    
    $reader = [System.IO.streamreader]$fs.Item('Zone.Identifier').open()
    try {
       do { 
          $line = $reader.ReadLine()
       } until (!$line -OR $line -eq '[ZoneTransfer]')
       $out = new-object PSObject
       while($zone = $reader.ReadLine()) {
          $zone = $zone -split "="
          if($zone.Count -lt 2) { break }
          Add-Member -in $out -Type NoteProperty -Name $zone[0] -value ([ZoneIdentifier]$zone[1])
       }
       $out
    } finally {
       $reader.close()
    }
 }
 }
 
 
 function Test-DownloadFlag {
 <#
 .Synopsis
 	Verify whether the ZoneTransfer flag is set (marking this file as one downloaded from the internet).
 .Description
 	Reads the Zone.Identifier alternate data stream (on NTFS file systems)
 .Parameter Path
 	The file you wish to check the ZoneTransfer flag on
 #>
 [CmdletBinding()]
 PARAM (
    [Parameter(Position=0, Mandatory=$true, ValueFromPipelineByPropertyName=$true)]
    [Alias("FullName")]
    [string]
    $Path
 )
 Process { 
    Get-ChildItem $Path | Select Name, @{N="Downloaded";E={ [bool]((new-object NTFS.FileStreams($_)).Item('Zone.Identifier')) } }, FullName, Length
 }
 }
 
 
 function Normalize-StreamName {
    PARAM($Path,$StreamName)
    if(!$StreamName -and !(Test-Path $Path -EA 0)) { 
       $Path, $Segment, $StreamName = $Path -split ":"
       if($StreamName -or (Test-Path ($Path,$Segment -join ":") -EA 0)) {
          $Path = $Path,$Segment -join ":" 
       } else {
          $StreamName = $Segment
       }
    }
    return $Path,$StreamName
 }
 
 
 function Get-Stream {
 <#
 .Synopsis
 	Get the list of alternate NTFS Streams
 .Description
 	Reads the named alternate data stream on NTFS file systems.
 .Parameter Path
 	The file you wish to read from. You may include the stream name in the format:
    C:\Path\File.extension:stream name
 .Parameter Stream
    The name of the stream you wish to read from. If you pass this as a separate parameter, you should NOT include it in the Path.
 #>
 [CmdletBinding()]
 PARAM (
    [Parameter(Position=0, Mandatory=$true, ValueFromPipelineByPropertyName=$true)]
    [Alias("FullName")]
    [string]
    $Path
 ,
    [Parameter(Position=1,Mandatory=$false)]
    [Alias("Name")][string]$StreamName  
 ,
    [Parameter()]
    [Switch]$Force
 )
 Process { 
    $Path,$Stream = Normalize-StreamName $Path $StreamName
    
    Write-Verbose "Path: $Path"
    Write-Verbose "Stream: $Stream"
    ForEach($file in Get-ChildItem $Path) {
       $FS = new-object NTFS.FileStreams($file)
       Write-Verbose "File: $File"
    
       if(!$Stream) {
          $FS
       } else {
          $FS | Where { $_.StreamName -like $Stream } | Tee -Var Output
          if($Force -and -not $Output) {
             $FS.add($Stream) > $null
             $FS.Item($Stream)
          }
       }
    }
 }
 }
 
 function Get-StreamContent {
 <#
 .Synopsis
 	Get the contents of a named NTFS Stream
 .Description
 	Reads the named alternate data stream (on NTFS file systems)
 .Parameter StreamInfo
    A StreamInfo object for the stream you want to get the content of.
 .Parameter Path
 	The file to read from. You may include the stream name in the format:
    C:\Path\File.extension:stream name
 .Parameter StreamName
    The name of the stream you wish to read from. If you pass this as a separate parameter, you should NOT include it in the Path.
 #>
 [CmdletBinding(DefaultParameterSetName="ByStream")]
 PARAM (
    [Parameter(Position=0, Mandatory=$true, ValueFromPipelineByPropertyName=$true, ParameterSetName="ByName")]
    [Alias("FullName")][string]$Path
 ,
    [Parameter(Position=0, Mandatory=$true, ValueFromPipeline=$true, ParameterSetName="ByStream")]
    [NTFS.StreamInfo]$StreamInfo
 ,
    [Parameter(Position=1, Mandatory=$false, ParameterSetName="ByName")]
    [Alias("Name")][string]$StreamName
 )
 Process {
    switch($PSCmdlet.ParameterSetName) {
       "ByName" {
          Get-Stream $Path $StreamName | Get-StreamContent
       }
       "ByStream" {
          $fileStream = $StreamInfo.open()
          $reader = [System.IO.StreamReader]$fileStream
          $reader.ReadToEnd()
          $fileStream.close()
       }
    }
 }
 }
 
 function Remove-Stream {
 <#
 .Synopsis
 	Remove a stream from a file (or, delete the file).
 .Description
 	Deletes the named alternate data stream (on NTFS file systems)
 .Parameter StreamInfo
    A StreamInfo object for the stream you want to get the content of.
 .Parameter Path
 	The file to delete from. You may include the stream name in the format:
    "C:\Path\File.extension:stream name"
 .Parameter StreamName
    The name of the stream you wish to remove. If you pass this as a separate parameter, you should NOT include it in the Path.
 #>
 [CmdletBinding(DefaultParameterSetName="ByStream")]
 PARAM (
    [Parameter(Position=0, Mandatory=$true, ValueFromPipelineByPropertyName=$true, ParameterSetName="ByName")]
    [Alias("FullName")][string]$Path
 ,
    [Parameter(Position=0, Mandatory=$true, ValueFromPipeline=$true, ParameterSetName="ByStream")]
    [NTFS.StreamInfo]$StreamInfo
 ,
    [Parameter(Position=1, Mandatory=$false, ParameterSetName="ByName")]
    [Alias("Name")][string]$StreamName
 )
 Process {
    switch($PSCmdlet.ParameterSetName) {
       "ByName" {
          foreach($StreamInfo in Get-Stream $Path $StreamName) {
             Write-Verbose $($StreamInfo |Out-String)
             $StreamInfo.Delete() > $null
          }
       }
       "ByStream" {
          $StreamInfo.Delete() > $null
       }
    }
 }
 }
 
 
 function Set-StreamContent {
 <#
 .Synopsis
 	Set the contents of a named NTFS Stream
 .Description
 	Sets the content of the named alternate data stream (on NTFS file systems)
 .Parameter StreamInfo
    A StreamInfo object for the stream you want to set the content of.
 .Parameter Path
 	The file to set content on. You may include the stream name in the format:
    "C:\Path\File.extension:stream name"
 .Parameter StreamName
    The name of the stream you wish to set. If you pass this as a separate parameter, you should NOT include it in the Path.
 #>
 [CmdletBinding(DefaultParameterSetName="ByStream")]
 PARAM (
    [Parameter(Position=0, Mandatory=$true, ValueFromPipelineByPropertyName=$true, ParameterSetName="ByName")]
    [Alias("FullName")][string]$Path
 ,
    [Parameter(Position=0, Mandatory=$true, ValueFromPipeline=$true, ParameterSetName="ByStream")]
    [NTFS.StreamInfo]$StreamInfo
 ,
    [Parameter(Position=1, Mandatory=$false, ParameterSetName="ByName")]
    [Alias("Name")][string]$StreamName
 , 
    [Parameter(Position=2, Mandatory=$true)]
    [String]$Value
 )
 Process {
    switch($PSCmdlet.ParameterSetName) {
       "ByName" {
          Write-Verbose "Path: $Path"
          Get-Stream $Path $StreamName -Force | Set-StreamContent -Value $Value
       }
       "ByStream" {
          $writer =[System.IO.streamwriter] $StreamInfo.Open()
          $writer.Write($value)
          $writer.close()
       }
    }
 }
 }
 
 
 Add-Type -TypeDefinition @'
 public enum ZoneIdentifier {
    Trusted = 1,
    Intranet = 2,
    Internet = 3,
    Untrusted = 4
 }
 '@
 
 
 Add-Type -TypeDefinition @'
 using System;
 using System.IO;
 using System.Collections;
 using System.Runtime.InteropServices;
 using Microsoft.Win32.SafeHandles;
 
 
 ///<summary>
 ///Encapsulates access to alternative data streams of an NTFS file.
 ///Adapted from a C++ sample by Dino Esposito,
 ///http://msdn.microsoft.com/library/default.asp?url=/library/en-us/dnfiles/html/ntfs5.asp
 ///</summary>
 namespace NTFS {
    /// <summary>
    /// Wraps the API functions, structures and constants.
    /// </summary>
    internal class Kernel32 
    {
       public const char STREAM_SEP = ':';
       public const int INVALID_HANDLE_VALUE = -1;
       public const int MAX_PATH = 256;
       
       [Flags()] public enum FileFlags : uint
       {
          WriteThrough = 0x80000000,
          Overlapped = 0x40000000,
          NoBuffering = 0x20000000,
          RandomAccess = 0x10000000,
          SequentialScan = 0x8000000,
          DeleteOnClose = 0x4000000,
          BackupSemantics = 0x2000000,
          PosixSemantics = 0x1000000,
          OpenReparsePoint = 0x200000,
          OpenNoRecall = 0x100000
       }
 
       [Flags()] public enum FileAccessAPI : uint
       {
          GENERIC_READ = 0x80000000,
          GENERIC_WRITE = 0x40000000
       }
       /// <summary>
       /// Provides a mapping between a System.IO.FileAccess value and a FileAccessAPI value.
       /// </summary>
       /// <param name="Access">The <see cref="System.IO.FileAccess"/> value to map.</param>
       /// <returns>The <see cref="FileAccessAPI"/> value.</returns>
       public static FileAccessAPI Access2API(FileAccess Access) 
       {
          FileAccessAPI lRet = 0;
          if ((Access & FileAccess.Read)==FileAccess.Read) lRet |= FileAccessAPI.GENERIC_READ;
          if ((Access & FileAccess.Write)==FileAccess.Write) lRet |= FileAccessAPI.GENERIC_WRITE;
          return lRet;
       }
 
       [StructLayout(LayoutKind.Sequential)] public struct LARGE_INTEGER 
       {
          public int Low;
          public int High;
 
          public long ToInt64() 
          {
             return (long)High * 4294967296 + (long)Low;
          }
       }
 
       [StructLayout(LayoutKind.Sequential)] public struct WIN32_STREAM_ID 
       {
          public int dwStreamID;
          public int dwStreamAttributes;
          public LARGE_INTEGER Length;
          public int dwStreamNameLength;
       }
       
       [DllImport("kernel32")] public static extern SafeFileHandle CreateFile(string Name, FileAccessAPI Access, FileShare Share, int Security, FileMode Creation, FileFlags Flags, int Template);
       [DllImport("kernel32")] public static extern bool DeleteFile(string Name);
       [DllImport("kernel32")] public static extern bool CloseHandle(SafeFileHandle hObject);
 
       [DllImport("kernel32")] public static extern bool BackupRead(SafeFileHandle hFile, IntPtr pBuffer, int lBytes, ref int lRead, bool bAbort, bool bSecurity, ref int Context);
       [DllImport("kernel32")] public static extern bool BackupRead(SafeFileHandle hFile, ref WIN32_STREAM_ID pBuffer, int lBytes, ref int lRead, bool bAbort, bool bSecurity, ref int Context);
       [DllImport("kernel32")] public static extern bool BackupSeek(SafeFileHandle hFile, int dwLowBytesToSeek, int dwHighBytesToSeek, ref int dwLow, ref int dwHigh, ref int Context);
    }
 
    /// <summary>
    /// Encapsulates a single alternative data stream for a file.
    /// </summary>
    public class StreamInfo 
    {
       private FileStreams _parent;
       private string _name;
       private long _length;
 
       internal StreamInfo(FileStreams Parent, string Name, long Length) 
       {
          _parent = Parent;
          _name = Name;
          _length = Length;
       }
 
       /// <summary>
       /// The name of the file.
       /// </summary>
       public string FileName 
       {
          get { return System.IO.Path.GetFileName(_parent.FileName); }
       }
       
       /// <summary>
       /// The name of the stream.
       /// </summary>
       public string StreamName 
       {
          get { return _name; }
       }
       
       /// <summary>
       /// The length (in bytes) of the stream.
       /// </summary>
       public long Length 
       {
          get { return _length; }
       }
       
       public override string ToString() 
       {
 		 if(String.IsNullOrEmpty(_name)) {
 			return _parent.FileName;
 		 } else {
 			return String.Format("{1}{0}{2}", Kernel32.STREAM_SEP, _parent.FileName, _name);
 		 }
       }
 
       public override bool Equals(Object o) 
       {
          if (o is StreamInfo) 
          {
             StreamInfo f = (StreamInfo)o;
             return (f._name.Equals(_name) && f._parent.Equals(_parent));
          }
          else if (o is string) 
          {
             return ((string)o).Equals(ToString());
          }
          else
             return base.Equals(o);
       }
       public override int GetHashCode() 
       {
          return ToString().GetHashCode();
       }
 
       /// <summary>
       /// Opens or creates the stream in read-write mode, with no sharing.
       /// </summary>
       /// <returns>A <see cref="System.IO.FileStream"/> wrapper for the stream.</returns>
       public FileStream Open() 
       {
          return Open(FileMode.OpenOrCreate, FileAccess.ReadWrite, FileShare.None);
       }
       /// <summary>
       /// Opens or creates the stream in read-write mode with no sharing.
       /// </summary>
       /// <param name="Mode">The <see cref="System.IO.FileMode"/> action for the stream.</param>
       /// <returns>A <see cref="System.IO.FileStream"/> wrapper for the stream.</returns>
       public FileStream Open(FileMode Mode) 
       {
          return Open(Mode, FileAccess.ReadWrite, FileShare.None);
       }
       /// <summary>
       /// Opens or creates the stream with no sharing.
       /// </summary>
       /// <param name="Mode">The <see cref="System.IO.FileMode"/> action for the stream.</param>
       /// <param name="Access">The <see cref="System.IO.FileAccess"/> level for the stream.</param>
       /// <returns>A <see cref="System.IO.FileStream"/> wrapper for the stream.</returns>
       public FileStream Open(FileMode Mode, FileAccess Access) 
       {
          return Open(Mode, Access, FileShare.None);
       }
       /// <summary>
       /// Opens or creates the stream.
       /// </summary>
       /// <param name="Mode">The <see cref="System.IO.FileMode"/> action for the stream.</param>
       /// <param name="Access">The <see cref="System.IO.FileAccess"/> level for the stream.</param>
       /// <param name="Share">The <see cref="System.IO.FileShare"/> level for the stream.</param>
       /// <returns>A <see cref="System.IO.FileStream"/> wrapper for the stream.</returns>
       public FileStream Open(FileMode Mode, FileAccess Access, FileShare Share) 
       {
          try 
          {
 			if(String.IsNullOrEmpty(_name)) {
 				return new FileStream(ToString(), Mode, Access, Share);
 			} else {
 				SafeFileHandle hFile = Kernel32.CreateFile(ToString() + Kernel32.STREAM_SEP + "$DATA", Kernel32.Access2API(Access), Share, 0, Mode, 0, 0);
 				return new FileStream(hFile, Access);
 			}
          }
          catch 
          {
             return null;
          }
       }
 
       /// <summary>
       /// Deletes the stream from the file.
       /// </summary>
       /// <returns>A <see cref="System.Boolean"/> value: true if the stream was deleted, false if there was an error.</returns>
       public bool Delete() 
       {
          return Kernel32.DeleteFile(ToString());
       }
    }
 
 
    /// <summary>
    /// Encapsulates the collection of alternative data streams for a file.
    /// A collection of <see cref="StreamInfo"/> objects.
    /// </summary>
    public class FileStreams : CollectionBase 
    {
       private FileInfo _file;
 
       public FileStreams(string File) 
       {
          _file = new FileInfo(File);
          initStreams();
       }
       public FileStreams(FileInfo file) 
       {
          _file = file;
          initStreams();
       }
 
       /// <summary>
       /// Reads the streams from the file.
       /// </summary>
       private void initStreams() 
       {
 		 base.List.Add(new StreamInfo(this,String.Empty,_file.Length));
 		 
          //Open the file with backup semantics
          SafeFileHandle hFile = Kernel32.CreateFile(_file.FullName, Kernel32.FileAccessAPI.GENERIC_READ, FileShare.Read, 0, FileMode.Open, Kernel32.FileFlags.BackupSemantics, 0);
          if (hFile.IsInvalid) return;
 
          try 
          {
             Kernel32.WIN32_STREAM_ID sid = new Kernel32.WIN32_STREAM_ID();
             int dwStreamHeaderLength = Marshal.SizeOf(sid);
             int Context = 0;
             bool Continue = true;
             while (Continue) 
             {
                //Read the next stream header
                int lRead = 0;
                Continue = Kernel32.BackupRead(hFile, ref sid, dwStreamHeaderLength, ref lRead, false, false, ref Context);
                if (Continue && lRead == dwStreamHeaderLength) 
                {
                   if (sid.dwStreamNameLength>0) 
                   {
                      //Read the stream name
                      lRead = 0;
                      IntPtr pName = Marshal.AllocHGlobal(sid.dwStreamNameLength);
                      try 
                      {
                         Continue = Kernel32.BackupRead(hFile, pName, sid.dwStreamNameLength, ref lRead, false, false, ref Context);
                         char[] bName = new char[sid.dwStreamNameLength];
                         Marshal.Copy(pName,bName,0,sid.dwStreamNameLength);
 
                         //Name is of the format ":NAME:$DATA\0"
                         string sName = new string(bName);
                         int i = sName.IndexOf(Kernel32.STREAM_SEP, 1);
                         if (i>-1) sName = sName.Substring(1, i-1);
                         else 
                         {
                            //This should never happen. 
                            //Truncate the name at the first null char.
                            i = sName.IndexOf('\0');
                            if (i>-1) sName = sName.Substring(1, i-1);
                         }
 
                         //Add the stream to the collection
                         base.List.Add(new StreamInfo(this,sName,sid.Length.ToInt64()));
                      }
                      finally 
                      {
                         Marshal.FreeHGlobal(pName);
                      }
                   }
 
                   //Skip the stream contents
                   int l = 0; int h = 0;
                   Continue = Kernel32.BackupSeek(hFile, sid.Length.Low, sid.Length.High, ref l, ref h, ref Context);
                }
                else break;
             }
          }
          finally 
          {
             Kernel32.CloseHandle(hFile);
          }
       }
 
       /// <summary>
       /// Returns the <see cref="System.IO.FileInfo"/> object for the wrapped file. 
       /// </summary>
       public FileInfo FileInfo 
       {
          get { return _file; }
       }
       /// <summary>
       /// Returns the full path to the wrapped file.
       /// </summary>
       public string FileName 
       {
          get { return _file.FullName; }
       }
 
       /// <summary>
       /// Returns the length of the main data stream, in bytes.
       /// </summary>
       public long Length {
          get {return _file.Length;}
       }
 
       /// <summary>
       /// Returns the length of all streams for the file, in bytes.
       /// </summary>
       public long FullLength
       {
          get 
          {	// don't initialize with "this.Length" anymore, because we include the default stream now
             long length = 0; 
             foreach (StreamInfo s in this) length += s.Length;
             return length;
          }
       }
 
       /// <summary>
       /// Opens or creates the default file stream.
       /// </summary>
       /// <returns><see cref="System.IO.FileStream"/></returns>
       public FileStream Open() 
       {
          return new FileStream(_file.FullName, FileMode.OpenOrCreate);
       }
 
       /// <summary>
       /// Opens or creates the default file stream.
       /// </summary>
       /// <param name="Mode">The <see cref="System.IO.FileMode"/> for the stream.</param>
       /// <returns><see cref="System.IO.FileStream"/></returns>
       public FileStream Open(FileMode Mode) 
       {
          return new FileStream(_file.FullName, Mode);
       }
 
       /// <summary>
       /// Opens or creates the default file stream.
       /// </summary>
       /// <param name="Mode">The <see cref="System.IO.FileMode"/> for the stream.</param>
       /// <param name="Access">The <see cref="System.IO.FileAccess"/> for the stream.</param>
       /// <returns><see cref="System.IO.FileStream"/></returns>
       public FileStream Open(FileMode Mode, FileAccess Access) 
       {
          return new FileStream(_file.FullName, Mode, Access);
       }
 
       /// <summary>
       /// Opens or creates the default file stream.
       /// </summary>
       /// <param name="Mode">The <see cref="System.IO.FileMode"/> for the stream.</param>
       /// <param name="Access">The <see cref="System.IO.FileAccess"/> for the stream.</param>
       /// <param name="Share">The <see cref="System.IO.FileShare"/> for the stream.</param>
       /// <returns><see cref="System.IO.FileStream"/></returns>
       public FileStream Open(FileMode Mode, FileAccess Access, FileShare Share) 
       {
          return new FileStream(_file.FullName, Mode, Access, Share);
       }
 
       /// <summary>
       /// Deletes the file, and all alternative streams.
       /// </summary>
       public void Delete() 
       {
          for (int i=base.List.Count;i>0;i--) 
          {
             base.List.RemoveAt(i);
          }
          _file.Delete();
       }
 
       /// <summary>
       /// Add an alternative data stream to this file.
       /// </summary>
       /// <param name="Name">The name for the stream.</param>
       /// <returns>The index of the new item.</returns>
       public int Add(string Name) 
       {
          StreamInfo FSI = new StreamInfo(this, Name, 0);
          int i = base.List.IndexOf(FSI);
          if (i==-1) i = base.List.Add(FSI);
          return i;
       }
       /// <summary>
       /// Removes the alternative data stream with the specified name.
       /// </summary>
       /// <param name="Name">The name of the string to remove.</param>
       public void Remove(string Name) 
       {
          StreamInfo FSI = new StreamInfo(this, Name, 0);
          int i = base.List.IndexOf(FSI);
          if (i>-1) base.List.RemoveAt(i);
       }
 
       /// <summary>
       /// Returns the index of the specified <see cref="StreamInfo"/> object in the collection.
       /// </summary>
       /// <param name="FSI">The object to find.</param>
       /// <returns>The index of the object, or -1.</returns>
       public int IndexOf(StreamInfo FSI) 
       {
          return base.List.IndexOf(FSI);
       }
       /// <summary>
       /// Returns the index of the <see cref="StreamInfo"/> object with the specified name in the collection.
       /// </summary>
       /// <param name="Name">The name of the stream to find.</param>
       /// <returns>The index of the stream, or -1.</returns>
       public int IndexOf(string Name) 
       {
          return base.List.IndexOf(new StreamInfo(this, Name, 0));
       }
 
       public StreamInfo this[int Index] 
       {
          get { return (StreamInfo)base.List[Index]; }
       }
       public StreamInfo this[string Name] 
       {
          get 
          { 
             int i = IndexOf(Name);
             if (i==-1) return null;
             else return (StreamInfo)base.List[i];
          }
       }
 
       /// <summary>
       /// Throws an exception if you try to add anything other than a StreamInfo object to the collection.
       /// </summary>
       protected override void OnInsert(int index, object value) 
       {
          if (!(value is StreamInfo)) throw new InvalidCastException();
       }
       /// <summary>
       /// Throws an exception if you try to add anything other than a StreamInfo object to the collection.
       /// </summary>
       protected override void OnSet(int index, object oldValue, object newValue) 
       {
          if (!(newValue is StreamInfo)) throw new InvalidCastException();
       }
 
       /// <summary>
       /// Deletes the stream from the file when you remove it from the list.
       /// </summary>
       protected override void OnRemoveComplete(int index, object value) 
       {
          try 
          {
             StreamInfo FSI = (StreamInfo)value;
             if (FSI != null) FSI.Delete();
          }
          catch {}
       }
 
       public new StreamEnumerator GetEnumerator() 
       {
          return new StreamEnumerator(this);
       }
 
       public class StreamEnumerator : object, IEnumerator 
       {
          private IEnumerator baseEnumerator;
             
          public StreamEnumerator(FileStreams mappings) 
          {
             this.baseEnumerator = ((IEnumerable)(mappings)).GetEnumerator();
          }
             
          public StreamInfo Current 
          {
             get 
             {
                return ((StreamInfo)(baseEnumerator.Current));
             }
          }
             
          object IEnumerator.Current 
          {
             get 
             {
                return baseEnumerator.Current;
             }
          }
             
          public bool MoveNext() 
          {
             return baseEnumerator.MoveNext();
          }
             
          bool IEnumerator.MoveNext() 
          {
             return baseEnumerator.MoveNext();
          }
             
          public void Reset() 
          {
             baseEnumerator.Reset();
          }
             
          void IEnumerator.Reset() 
          {
             baseEnumerator.Reset();
          }
       }
    }
 }
 '@
 
 
 Set-Alias block Set-DownloadFlag
 Set-Alias unblock Remove-DownloadFlag
 
 Export-ModuleMember -Function Remove-DownloadFlag, Set-DownloadFlag, Get-DownloadFlag, Test-DownloadFlag, 
    Get-Stream, Get-StreamContent, Remove-Stream, Set-StreamContent -alias block, unblock
`

