---
Author: skourlatov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5398
Published Date: 2015-09-04t02
Archived Date: 2015-01-31t20
---

# get-fileallocation - 

## Description

how to obtain file fragmentation data via powershell? that’s answer.

## Comments



## Usage



## TODO



## function

`get-fileallocation`

## Code

`#
 #
 Function Get-FileAllocation
 {
     param
     (
         [parameter(Position=0,ValueFromPipeline=$true,Mandatory=$true)]
         [string]$FilePath
     )
 
     try
     {
         $file = $FilePath | Get-Item -Force -ea 'Stop'
     }
     catch
     {
         throw "Invalid path"
     }
 
     {
         throw "The file is a reparse point"
     }
     return [PoshCode.FileSystem]::GetFileAllocation($file.FullName)
 }
 
 Add-Type -TypeDefinition @"
 using System;
 using System.ComponentModel;
 using System.Collections.Generic;
 using System.Runtime.InteropServices;
 using Microsoft.Win32.SafeHandles;
 
 namespace PoshCode
 {
     public class FileSystem
     {
         [DllImport("kernel32.dll", SetLastError = true, CharSet = CharSet.Auto)]
         private static extern SafeFileHandle CreateFile(
                                           string lpFileName,
             [MarshalAs(UnmanagedType.U4)] uint dwDesiredAccess,
             [MarshalAs(UnmanagedType.U4)] uint dwShareMode,
                                           IntPtr lpSecurityAttributes,
             [MarshalAs(UnmanagedType.U4)] uint dwCreationDisposition,
             [MarshalAs(UnmanagedType.U4)] uint dwFlagsAndAttributes,
                                           IntPtr hTemplateFile);
 
         private const int FSCTL_GET_RETRIEVAL_POINTERS = 0x00090073;
         private const uint FileAccess_GenericRead  = 0x80000000;
         private const uint FileAccess_GenericWrite = 0x40000000;
         private const uint FileShare_Read          = 0x00000001;
         private const uint FileShare_Write         = 0x00000002;
         private const uint FileShare_ReadWrite     = FileShare_Read | FileShare_Write;
         private const uint FileMode_OpenExisting   = 0x00000003;
         private const uint FileAttributes_Normal   = 0x00000080;
 
         private static SafeFileHandle GetSafeHandle(string path, bool allowrite)
         {
             var PreferredAccessMode = FileAccess_GenericRead;
             if (allowrite)
                 PreferredAccessMode |= FileAccess_GenericWrite;
             return CreateFile(path, PreferredAccessMode, FileShare_ReadWrite, IntPtr.Zero,
                 FileMode_OpenExisting, FileAttributes_Normal, IntPtr.Zero);
         }
 
         [StructLayout(LayoutKind.Sequential)]
         private struct StartingVcnInputBuffer
         {
             public static readonly int Size;
             static StartingVcnInputBuffer() { Size = Marshal.SizeOf(typeof(StartingVcnInputBuffer)); }
             public long StartingVcn;
         }
         [StructLayout(LayoutKind.Sequential)]
         private struct RetrievalPointersBuffer
         {
             public static readonly int Size;
             static RetrievalPointersBuffer() { Size = Marshal.SizeOf(typeof(RetrievalPointersBuffer)); }
             public int ExtentCount;
             public long StartingVcn;
             // Extents
             public long NextVcn;
             public long Lcn;
         }
         [StructLayout(LayoutKind.Sequential)]
         public struct PhysicalFileFragment
         {
             public long Fragment;
             public long StartCluster;
             public long Length;
         }
         [StructLayout(LayoutKind.Sequential)]
         public struct Win32FileAllocationData
         {
             public uint TotalClusters;
             public uint TotalFragments;
             public List<PhysicalFileFragment> PhysicalAllocation;
         }
 
         [DllImport("kernel32.dll", SetLastError = true)]
         [return: MarshalAs(UnmanagedType.Bool)]
         private static extern bool DeviceIoControl(
             SafeFileHandle hFile,
             uint ioctl,
             ref StartingVcnInputBuffer invalue,
             int InSize,
             out RetrievalPointersBuffer outvalue,
             int OutSize,
             int BytesReturned,
             IntPtr zerovalue
         );
 
         private const int ERROR_HANDLE_EOF = 0x00000026;
         private const int ERROR_MORE_DATA  = 0x000000EA;
         private const int NO_ERROR         = 0x00000000;
         private static int lpBytesReturned = 0x00000000;
 
         public static Win32FileAllocationData GetFileAllocation(string path)
         {
             var vcnIn    = new StartingVcnInputBuffer();
             var rpbOut    = new RetrievalPointersBuffer();
             var alloc    = new Win32FileAllocationData();
             var frag    = new PhysicalFileFragment();
             long fragLength;
             bool doInc        = false;
             bool newFrag    = false;
             int extentNumber = 0;
             int ERR;
 
              vcnIn.StartingVcn    = 0L;
             alloc.TotalClusters    = 0;
             alloc.PhysicalAllocation = new List<PhysicalFileFragment>();
  
             // file open
             using (SafeFileHandle handle = GetSafeHandle(path, false))
             {    do
                 {
                     DeviceIoControl(
                         handle, FSCTL_GET_RETRIEVAL_POINTERS,
                         ref vcnIn, StartingVcnInputBuffer.Size, 
                         out rpbOut, RetrievalPointersBuffer.Size,
                         lpBytesReturned, IntPtr.Zero
                     );
  
                     ERR = Marshal.GetLastWin32Error();
                      switch (ERR)
                     {
                         case ERROR_HANDLE_EOF:
                             break;
                         case NO_ERROR:
                             doInc = true;
                             break;
                         case ERROR_MORE_DATA:
                             doInc = true;
                             vcnIn.StartingVcn = rpbOut.NextVcn;
                             break;
                         default:
                             throw new Win32Exception(ERR);
                     }
 
                     if (doInc && rpbOut.Lcn >= 0) // Some files may have dummy "pieces" - reject them
                     {
                         fragLength = rpbOut.NextVcn - rpbOut.StartingVcn;
                         alloc.TotalClusters += (uint)fragLength;
 
                         if (extentNumber == 0) // Process started - getting a new fragment
                         {
                             newFrag = true;
                         }
                         else
                         {
                             if (frag.StartCluster + frag.Length == rpbOut.Lcn) // No new fragment - "lengthen" existing
                             {
                                 frag.Length += fragLength;
                             }
                             else // There is a new fragment - adding existing to the list and getting new one
                             {
                                 alloc.PhysicalAllocation.Add(frag);
                                 newFrag = true;
                             }
                         }
 
                         if (newFrag) // Getting new fragment
                         {
                             frag.Fragment = extentNumber++;
                             frag.StartCluster = rpbOut.Lcn;
                             frag.Length = fragLength;
                             newFrag = false;
                         }
 
                         doInc = false;
                     }
 
                 } while (ERR == ERROR_MORE_DATA);
 
                 if (frag.Length > 0)
                     alloc.PhysicalAllocation.Add(frag); // Process ended - adding the last fragment to the list
                 alloc.TotalFragments = (uint)alloc.PhysicalAllocation.Count;
             }
             // file close
 
             return alloc;
         }
     }
 }
 "@
`

