---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 2.01
Encoding: ascii
License: cc0
PoshCode ID: 4100
Published Date: 2013-04-11t16
Archived Date: 2013-05-13t12
---

# filecop - 

## Description

what is a filecop? it’s a script that shows extended data of files such as hashes, streams and etc. in compact gui style. note that script has been tested on win2k3, so i do not have any idea about its workability on other systems. please, let me know about bugs.

## Comments



## Usage



## TODO



## function

`add-rootstree`

## Code

`#
 #
 $nul = "<NULL>"
 $sec = "MD5", "SHA1", "SHA256", "SHA384", "SHA512", "RIPEMD160"
 
 ##################################################################################################
 
 function Add-RootsTree {
   [IO.Directory]::GetLogicalDrives() | % {
     $nod = New-Object Windows.Forms.TreeNode
     $nod = $tvRoots.Nodes.Add($_)
     $nod.Nodes.Add($nul)
   }
 }
 
 function Show-ExceptMessage([string]$msg) {
   [Windows.Forms.MessageBox]::Show($msg, $frmMain.Text,
                  [Windows.Forms.MessageBoxButtons]::OK,
             [Windows.Forms.MessageBoxIcon]::Exclamation
   )
 }
 
 function Add-Folders {
   $_.Node.Nodes.Clear()
 
   try {
     foreach ($i in [IO.Directory]::GetDirectories($_.Node.FullPath)) {
       $node = $_.Node.Nodes.Add([IO.Path]::GetFileName($i))
       $node.Nodes.Add($nul)
     }
   }
   catch { Show-ExceptMessage $_.Exception.Message }
 }
 
 function Add-Files {
   try {
     foreach ($i in [IO.Directory]::GetFiles($_.Node.FullPath)) {
       $node = $_.Node.Nodes.Add([IO.Path]::GetFileName($i))
       $node.Tag = "File"
       $node.ImageIndex = 1
     }
   }
   catch {}
 }
 
 function Format-String([string]$par) {
   if ($par -eq "") { 'n/a' }
   else { $par }
 }
 
 function Get-Checksum([string]$alg, [string]$obj) {
   ([Security.Cryptography.HashAlgorithm]::Create($alg).ComputeHash(
     [IO.File]::ReadAllBytes($obj)
   ) | % {"{0:x2}" -f $_}) -as [string] -replace " ", ""
 }
 
 function Get-Image([string]$img) {
   [Drawing.Image]::FromStream((New-Object IO.MemoryStream(($$ = `
               [Convert]::FromBase64String($img)), 0, $$.Length)))
 }
 
 function Clear-Output {
   $lblName.Text = [String]::Empty
   $lblAttr.Text = [String]::Empty
   $lblSign.Text = [String]::Empty
   $lblBorn.Text = [String]::Empty
   $lblAccs.Text = [String]::Empty
   $lblWrit.Text = [String]::Empty
   $lblSize.Text = [String]::Empty
   $lblType.Text = [String]::Empty
   $lblPubl.Text = [String]::Empty
   $lblDesc.Text = [String]::Empty
   $lblProd.Text = [String]::Empty
   $lblVers.Text = [String]::Empty
   $lblOriN.Text = [String]::Empty
   $lblIntN.Text = [String]::Empty
   $lblCopy.Text = [String]::Empty
   $lblComm.Text = [String]::Empty
 }
 
 ##################################################################################################
 
 $tvRoots_AfterSelect= {
   $lstStrm.Items.Clear()
   $lstHash.Items.Clear()
 
   if ($_.Node.Tag -eq 'File') {
     $fi = [IO.FileInfo]($_.Node.FullPath)
     if (Test-Path $fi.FullName) {
       $lblName.Text = $fi.FullName
       $lblAttr.Text = $fi.Attributes
       $lblSign.Text = (Get-AuthenticodeSignature $fi.FullName).Status
       $lblBorn.Text = $fi.CreationTime
       $lblAccs.Text = $fi.LastAccessTime
       $lblWrit.Text = $fi.LastWriteTime
       $lblSize.Text = $fi.Length
       $lblType.Text = (New-Object -com Scripting.FileSystemObject).GetFile($fi.FullName).Type
 
       $vi = [Diagnostics.FileVersionInfo]::GetVersionInfo($fi.FullName)
       $lblPubl.Text = (Format-String $vi.CompanyName)
       $lblDesc.Text = (Format-String $vi.FileDescription)
       $lblProd.Text = (Format-String $vi.ProductName)
       $lblVers.Text = (Format-String $vi.ProductVersion)
       $lblOriN.Text = (Format-String $vi.OriginalFileName)
       $lblIntN.Text = (Format-String $vi.InternalName)
       $lblCopy.Text = (Format-String $vi.LegalCopyright)
       $lblComm.Text = (Format-String $vi.Comments)
 
       try {
         [IO.Streams]::Dump($fi.FullName).Split(";") | % {
           $itm = $lstStrm.Items.Add($_.Split(" ")[0])
           $itm.SubItems.Add($_.Split(" ")[1])
           $itm.SubItems.Add($_.Split(" ")[2])
         }
       }
       catch {}
 
       $sec | % {
         $itm = $lstHash.Items.Add($_)
 
         if ($fi.Length -ne 0) {
           $itm.SubItems.Add((Get-Checksum $_ $fi.FullName))
         }
         else { $itm.SubItems.Add('n/a') }
       }
     }
     else { Clear-Output; $lblName.Text = 'File does not exist or has been moved' }
   }
   else { Clear-Output }
 }
 
 $tvRoots_BeforeExpand= {
   Add-Folders
   Add-Files
 }
 
 $mnuUpdt_Click= {
   $tvRoots.Nodes.Clear()
   $lstStrm.Items.Clear()
   $lstHash.Items.Clear()
   Clear-Output
   $sbPanel.Text = "Refreshed at " + (Get-Date -f 'HH:mm:ss')
   Add-RootsTree
 }
 
 $mnuSBar_Click= {
   $toggle =! $mnuSBar.Checked
   $mnuSbar.Checked = $toggle
   $sbPanel.Visible = $toggle
 }
 
 $frmMain_load= {
   Add-RootsTree
   $sbPanel.Text = "Ready"
 }
 
 ##################################################################################################
 
 $code = @'
 using System;
 using System.IO;
 using System.ComponentModel;
 using System.Collections.Generic;
 using Microsoft.Win32.SafeHandles;
 using System.Runtime.InteropServices;
 
 namespace IO {
   [StructLayout(LayoutKind.Sequential, Pack = 1)]
   struct Win32StreamID {
     public StreamType dwStreamId;
     public int dwStreamAttributes;
     public long Size;
     public int dwStreamNameSize;
   }
 
   public enum StreamType {
     Unknown = 0,
     Data,
     ExternalData,
     SecurityData,
     AlternateData,
     Link,
     PropertyData,
     ObjectId,
     ReparseData,
     SparseDock,
     TransactionData
   }
 
   struct StreamInfo {
     public StreamInfo(string name, StreamType type, long size) {
       Name = name;
       Type = type;
       Size = size;
     }
     public readonly string Name;
     public readonly StreamType Type;
     public readonly long Size;
   }
 
   internal static class WinAPI {
     [DllImport("kernel32.dll")]
     [return: MarshalAs(UnmanagedType.Bool)]
     private static extern bool BackupRead(SafeFileHandle hFile, IntPtr lpBuffer,
                         uint nNumberOfBytesToRead, out uint lpNumberOfBytesRead,
                                     [MarshalAs(UnmanagedType.Bool)] bool bAbort,
                           [MarshalAs(UnmanagedType.Bool)] bool bProcessSecurity,
                                                           ref IntPtr lpContext);
 
     [DllImport("kernel32.dll")]
     [return: MarshalAs(UnmanagedType.Bool)]
     private static extern bool BackupSeek(SafeFileHandle hFile, uint dwLowBytesToSeek,
                                    uint dwHighBytesToSeek, out uint lpdwLowByteSeeked,
                                    out uint lpdwHighByteSeeked, ref IntPtr lpContext);
 
     public static IEnumerable<StreamInfo> GetStreams(FileInfo file) {
       const int buff = 4096;
 
       using (FileStream fs = file.OpenRead()) {
         IntPtr context = IntPtr.Zero;
         IntPtr buffer = Marshal.AllocHGlobal(buff);
 
         try {
           while (true) {
             uint numRead;
             if (!BackupRead(fs.SafeFileHandle, buffer,
               (uint)Marshal.SizeOf(typeof(Win32StreamID)), out numRead,
               false, true, ref context)) throw new Win32Exception();
 
             if (numRead > 0) {
               Win32StreamID stream = (Win32StreamID)Marshal.PtrToStructure(buffer,
                                                            typeof(Win32StreamID));
               string name = null;
               if (stream.dwStreamNameSize > 0) {
                 if (!BackupRead(fs.SafeFileHandle, buffer, (uint)Math.Min(buff,
                                          stream.dwStreamNameSize), out numRead,
                                             false, true, ref context)) throw new Win32Exception();
                 name = Marshal.PtrToStringUni(buffer, (int)numRead / 2);
               }
 
               yield return new StreamInfo(name, stream.dwStreamId, stream.Size);
 
               if (stream.Size > 0) {
                 uint lo, hi;
                 BackupSeek(fs.SafeFileHandle, uint.MaxValue,
                  int.MaxValue, out lo, out hi, ref context);
               }
             }
             else break;
           }
         }
         finally {
           Marshal.FreeHGlobal(buffer);
           uint numRead;
           if (!BackupRead(fs.SafeFileHandle, IntPtr.Zero, 0,
                    out numRead, true, false, ref context)) throw new Win32Exception();
         }
       }
     }
   }
 
   public static class Streams {
     public static string Dump(string file) {
       List<StreamInfo> streams =
         new List<StreamInfo>(WinAPI.GetStreams(new FileInfo(file)));
 
       string res = "";
       foreach (StreamInfo stream in streams) {
         res += (stream.Name != null ? stream.Name : "<Unnamed>") +
                       " " + stream.Type + " " + stream.Size + ";";
       }
 
       return res;
     }
   }
 }
 '@
 
 ##################################################################################################
 
 function frmMain_Show {
   Add-Type -AssemblyName System.Windows.Forms
   [Windows.Forms.Application]::EnableVisualStyles()
 
   Add-Type $code
   $ico = [Drawing.Icon]::ExtractAssociatedIcon(($PSHome + '\powershell.exe'))
 
   $frmMain = New-Object Windows.Forms.Form
   $mnuMain = New-Object Windows.Forms.MainMenu
   $mnuFile = New-Object Windows.Forms.MenuItem
   $mnuUpdt = New-Object Windows.Forms.MenuItem
   $mnuExit = New-Object Windows.Forms.MenuItem
   $mnuView = New-Object Windows.Forms.MenuItem
   $mnuTCol = New-Object Windows.Forms.MenuItem
   $mnuSBar = New-Object Windows.Forms.MenuItem
   $mnuHelp = New-Object Windows.Forms.MenuItem
   $mnuInfo = New-Object Windows.Forms.MenuItem
   $scSplt1 = New-Object Windows.Forms.SplitContainer
   $scSplt2 = New-Object Windows.Forms.SplitContainer
   $tvRoots = New-Object Windows.Forms.TreeView
   $lblL_01 = New-Object Windows.Forms.Label
   $lblL_02 = New-Object Windows.Forms.Label
   $lblL_03 = New-Object Windows.Forms.Label
   $lblL_04 = New-Object Windows.Forms.Label
   $lblL_05 = New-Object Windows.Forms.Label
   $lblL_06 = New-Object Windows.Forms.Label
   $lblL_07 = New-Object Windows.Forms.Label
   $lblL_08 = New-Object Windows.Forms.Label
   $lblL_09 = New-Object Windows.Forms.Label
   $lblL_10 = New-Object Windows.Forms.Label
   $lblL_11 = New-Object Windows.Forms.Label
   $lblL_12 = New-Object Windows.Forms.Label
   $lblL_13 = New-Object Windows.Forms.Label
   $lblL_14 = New-Object Windows.Forms.Label
   $lblL_15 = New-Object Windows.Forms.Label
   $lblL_16 = New-Object Windows.Forms.Label
   $lblName = New-Object Windows.Forms.Label
   $lblAttr = New-Object Windows.Forms.Label
   $lblSign = New-Object Windows.Forms.Label
   $lblBorn = New-Object Windows.Forms.Label
   $lblAccs = New-Object Windows.Forms.Label
   $lblWrit = New-Object Windows.Forms.Label
   $lblSize = New-Object Windows.Forms.Label
   $lblType = New-Object Windows.Forms.Label
   $lblPubl = New-Object Windows.Forms.Label
   $lblDesc = New-Object Windows.Forms.Label
   $lblProd = New-Object Windows.Forms.Label
   $lblVers = New-Object Windows.Forms.Label
   $lblOriN = New-Object Windows.Forms.Label
   $lblIntN = New-Object Windows.Forms.Label
   $lblCopy = New-Object Windows.Forms.Label
   $lblComm = New-Object Windows.Forms.Label
   $lstStrm = New-Object Windows.Forms.ListView
   $chSName = New-Object Windows.Forms.ColumnHeader
   $chSType = New-Object Windows.Forms.ColumnHeader
   $chSSize = New-Object Windows.Forms.ColumnHeader
   $lstHash = New-Object Windows.Forms.ListView
   $chHType = New-Object Windows.Forms.ColumnHeader
   $chHData = New-Object Windows.Forms.ColumnHeader
   $imgList = New-Object Windows.Forms.ImageList
   $sbPanel = New-Object Windows.Forms.StatusBar
   #
   #
   $mnuMain.MenuItems.AddRange(@($mnuFile, $mnuView, $mnuHelp))
   #
   #
   $mnuFile.MenuItems.AddRange(@($mnuUpdt, $mnuExit))
   $mnuFile.Text = "&File"
   #
   #
   $mnuUpdt.Shortcut = "F5"
   $mnuUpdt.Text = "&Refresh"
   $mnuUpdt.Add_Click($mnuUpdt_Click)
   #
   #
   $mnuExit.Shortcut = "CtrlX"
   $mnuExit.Text = "E&xit"
   $mnuExit.Add_Click({$frmMain.Close()})
   #
   #
   $mnuView.MenuItems.AddRange(@($mnuTCol, $mnuSBar))
   $mnuView.Text = "&View"
   #
   #
   $mnuTCol.Shortcut = "CtrlC"
   $mnuTCol.Text = "&Collapse Tree"
   $mnuTCol.Add_Click({$tvRoots.CollapseAll()})
   #
   #
   $mnuSBar.Checked = $true
   $mnuSBar.Text = "Toggle Status &Bar"
   $mnuSBar.Add_Click($mnuSBar_Click)
   #
   #
   $mnuHelp.MenuItems.AddRange(@($mnuInfo))
   $mnuHelp.Text = "&Help"
   #
   #
   $mnuInfo.Text = "About..."
   $mnuInfo.Add_Click({frmInfo_Show})
   #
   #
   $scSplt1.Dock = "Fill"
   $scSplt1.Orientation = "Horizontal"
   $scSplt1.Panel1.Controls.Add($scSplt2)
   $scSplt1.Panel2.Controls.Add($lstHash)
   $scSplt1.Panel1MinSize = 450
   $scSplt1.SplitterDistance = 90
   $scSplt1.SplitterWidth = 1
   #
   #
   $scSplt2.Dock = "Fill"
   $scSplt2.Panel1.Controls.Add($tvRoots)
   $scSplt2.Panel2.Controls.AddRange(@($lblL_01, $lblL_02, $lblL_03, $lblL_04, $lblL_05, `
         $lblL_06, $lblL_07, $lblL_08, $lblL_09, $lblL_10, $lblL_11, $lblL_12, $lblL_13, `
         $lblL_14, $lblL_15, $lblL_16, $lblName, $lblAttr, $lblSign, $lblBorn, $lblAccs, `
         $lblWrit, $lblSize, $lblType, $lblPubl, $lblDesc, $lblProd, $lblVers, $lblOriN, `
         $lblIntN, $lblCopy, $lblComm, $lstStrm))
   $scSplt2.Panel1MinSize = 43
   $scSplt2.SplitterDistance = 43
   $scSplt2.SplitterWidth = 1
   #
   #
   $tvRoots.Dock = "Fill"
   $tvRoots.ImageList = $imgList
   $tvRoots.Select()
   $tvRoots.SelectedImageIndex = 2
   $tvRoots.Add_AfterSelect($tvRoots_AfterSelect)
   $tvRoots.Add_BeforeExpand($tvRoots_BeforeExpand)
   #
   #
   $lblL_01.Location = New-Object Drawing.Point(8, 13)
   $lblL_01.Width = 80
   $lblL_01.Text = "Full Name:"
   #
   #
   $lblL_02.Location = New-Object Drawing.Point(8, 35)
   $lblL_02.Width = 80
   $lblL_02.Text = "Attributes:"
   #
   #
   $lblL_03.Location = New-Object Drawing.Point(8, 57)
   $lblL_03.Width = 80
   $lblL_03.Text = "Signed:"
   #
   #
   $lblL_04.Location = New-Object Drawing.Point(8, 79)
   $lblL_04.Width = 80
   $lblL_04.Text = "Creation:"
   #
   #
   $lblL_05.Location = New-Object Drawing.Point(8, 101)
   $lblL_05.Width = 80
   $lblL_05.Text = "Last Access:"
   #
   #
   $lblL_06.Location = New-Object Drawing.Point(8, 123)
   $lblL_06.Width = 80
   $lblL_06.Text = "Last Write:"
   #
   #
   $lblL_07.Location = New-Object Drawing.Point(8, 145)
   $lblL_07.Width = 80
   $lblL_07.Text = "Size:"
   #
   #
   $lblL_08.Location = New-Object Drawing.Point(8, 167)
   $lblL_08.Width = 80
   $lblL_08.Text = "Shell Type:"
   #
   #
   $lblL_09.Location = New-Object Drawing.Point(8, 189)
   $lblL_09.Width = 80
   $lblL_09.Text = "Publisher:"
   #
   #
   $lblL_10.Location = New-Object Drawing.Point(8, 211)
   $lblL_10.Width = 80
   $lblL_10.Text = "Description:"
   #
   #
   $lblL_11.Location = New-Object Drawing.Point(8, 233)
   $lblL_11.Width = 80
   $lblL_11.Text = "Product:"
   #
   #
   $lblL_12.Location = New-Object Drawing.Point(8, 255)
   $lblL_12.Width = 80
   $lblL_12.Text = "Version:"
   #
   #
   $lblL_13.Location = New-Object Drawing.Point(8, 277)
   $lblL_13.Width = 80
   $lblL_13.Text = "Original Name:"
   #
   #
   $lblL_14.Location = New-Object Drawing.Point(8, 299)
   $lblL_14.Width = 80
   $lblL_14.Text = "Internal Name:"
   #
   #
   $lblL_15.Location = New-Object Drawing.Point(8, 321)
   $lblL_15.Width = 80
   $lblL_15.Text = "Copyright:"
   #
   #
   $lblL_16.Location = New-Object Drawing.Point(8, 343)
   $lblL_16.Width = 80
   $lblL_16.Text = "Comment:"
   #
   #
   $lblName.BorderStyle = "Fixed3D"
   $lblName.Location = New-Object Drawing.Point(91, 11)
   $lblName.Size = New-Object Drawing.Size(461, 20)
   $lblName.TextAlign = "MiddleLeft"
   #
   #
   $lblAttr.BorderStyle = "Fixed3D"
   $lblAttr.Location = New-Object Drawing.Point(91, 33)
   $lblAttr.Size = New-Object Drawing.Size(461, 20)
   $lblAttr.TextAlign = "MiddleLeft"
   #
   #
   $lblSign.BorderStyle = "Fixed3D"
   $lblSign.Location = New-Object Drawing.Point(91, 55)
   $lblSign.Size = New-Object Drawing.Size(461, 20)
   $lblSign.TextAlign = "MiddleLeft"
   #
   #
   $lblBorn.BorderStyle = "Fixed3D"
   $lblBorn.Location = New-Object Drawing.Point(91, 77)
   $lblBorn.Size = New-Object Drawing.Size(461, 20)
   $lblBorn.TextAlign = "MiddleLeft"
   #
   #
   $lblAccs.BorderStyle = "Fixed3D"
   $lblAccs.Location = New-Object Drawing.Point(91, 99)
   $lblAccs.Size = New-Object Drawing.Size(461, 20)
   $lblAccs.TextAlign = "MiddleLeft"
   #
   #
   $lblWrit.BorderStyle = "Fixed3D"
   $lblWrit.Location = New-Object Drawing.Point(91, 121)
   $lblWrit.Size = New-Object Drawing.Size(461, 20)
   $lblWrit.TextAlign = "MiddleLeft"
   #
   #
   $lblSize.BorderStyle = "Fixed3D"
   $lblSize.Location = New-Object Drawing.Point(91, 143)
   $lblSize.Size = New-Object Drawing.Size(461, 20)
   $lblSize.TextAlign = "MiddleLeft"
   #
   #
   $lblType.BorderStyle = "Fixed3D"
   $lblType.Location = New-Object Drawing.Point(91, 165)
   $lblType.Size = New-Object Drawing.Size(461, 20)
   $lblType.TextAlign = "MiddleLeft"
   #
   #
   $lblPubl.BorderStyle = "Fixed3D"
   $lblPubl.Location = New-Object Drawing.Point(91, 187)
   $lblPubl.Size = New-Object Drawing.Size(461, 20)
   $lblPubl.TextAlign = "MiddleLeft"
   #
   #
   $lblDesc.BorderStyle = "Fixed3D"
   $lblDesc.Location = New-Object Drawing.Point(91, 209)
   $lblDesc.Size = New-Object Drawing.Size(461, 20)
   $lblDesc.TextAlign = "MiddleLeft"
   #
   #
   $lblProd.BorderStyle = "Fixed3D"
   $lblProd.Location = New-Object Drawing.Point(91, 231)
   $lblProd.Size = New-Object Drawing.Size(461, 20)
   $lblProd.TextAlign = "MiddleLeft"
   #
   #
   $lblVers.BorderStyle = "Fixed3D"
   $lblVers.Location = New-Object Drawing.Point(91, 253)
   $lblVers.Size = New-Object Drawing.Size(461, 20)
   $lblVers.TextAlign = "MiddleLeft"
   #
   #
   $lblOriN.BorderStyle = "Fixed3D"
   $lblOriN.Location = New-Object Drawing.Point(91, 275)
   $lblOriN.Size = New-Object Drawing.Size(461, 20)
   $lblOriN.TextAlign = "MiddleLeft"
   #
   #
   $lblIntN.BorderStyle = "Fixed3D"
   $lblIntN.Location = New-Object Drawing.Point(91, 297)
   $lblIntN.Size = New-Object Drawing.Size(461, 20)
   $lblIntN.TextAlign = "MiddleLeft"
   #
   #
   $lblCopy.BorderStyle = "Fixed3D"
   $lblCopy.Location = New-Object Drawing.Point(91, 319)
   $lblCopy.Size = New-Object Drawing.Size(461, 20)
   $lblCopy.TextAlign = "MiddleLeft"
   #
   #
   #$lblComm.Anchor = "Left, Top, Right"
   $lblComm.BorderStyle = "Fixed3D"
   $lblComm.Location = New-Object Drawing.Point(91, 341)
   $lblComm.Size = New-Object Drawing.Size(461, 20)
   $lblComm.TextAlign = "MiddleLeft"
   #
   #
   $lstStrm.Columns.AddRange(@($chSName, $chSType, $chSSize))
   $lstStrm.FullRowSelect = $true
   $lstStrm.Location = New-Object Drawing.Point(8, 367)
   $lstStrm.Size = New-Object Drawing.Size(545, 80)
   $lstStrm.View = "Details"
   #
   #
   $chSName.Text = "Stream Name"
   $chSName.Width = 150
   #
   #
   $chSType.Text = "Stream Type"
   $chSType.TextAlign = "Right"
   $chSType.Width = 150
   #
   #
   $chSSize.Text = "Stream Size"
   $chSSize.Width = 75
   #
   #
   $lstHash.Columns.AddRange(@($chHType, $chHData))
   $lstHash.Dock = "Fill"
   $lstHash.FullRowSelect = $true
   $lstHash.ShowItemToolTips = $true
   $lstHash.View = "Details"
   #
   #
   $chHType.Text = "Hash"
   $chHType.TextAlign = "Right"
   $chHType.Width = 90
   #
   #
   $chHData.Text = "Data"
   $chHData.Width = 679
   #
   #
   $i_1, $i_2, $i_3 | % { $imgList.Images.Add((Get-Image $_)) }
   #
   #
   $sbPanel.SizingGrip = $false
   #
   #
   $frmMain.ClientSize = New-Object Drawing.Size(790, 550)
   $frmMain.Controls.AddRange(@($scSplt1, $sbPanel))
   $frmMain.FormBorderStyle = "FixedSingle"
   $frmMain.Icon = $ico
   $frmMain.MaximizeBox = $false
   $frmMain.Menu = $mnuMain
   $frmMain.StartPosition = "CenterScreen"
   $frmMain.Text = "FileCop [" + [Security.Principal.WindowsIdentity]::GetCurrent().Name + "]"
   $frmMain.Add_Load($frmMain_Load)
 
   [void]$frmMain.ShowDialog()
 }
 
 ##################################################################################################
 
 function frmInfo_Show {
   $frmInfo = New-Object Windows.Forms.Form
   $pbImage = New-Object Windows.Forms.PictureBox
   $lblName = New-Object Windows.Forms.Label
   $lblCopy = New-Object Windows.Forms.Label
   $btnExit = New-Object Windows.Forms.Button
   #
   #
   $pbImage.Location = New-Object Drawing.Point(16, 16)
   $pbImage.Size = New-Object Drawing.Size(32, 32)
   $pbImage.SizeMode = "StretchImage"
   #
   #
   $lblName.Font = New-Object Drawing.Font("Microsoft Sans Serif", 8, [Drawing.FontStyle]::Bold)
   $lblName.Location = New-Object Drawing.Point(53, 19)
   $lblName.Size = New-Object Drawing.Size(360, 18)
   $lblName.Text = "FileCop v2.01"
   #
   #
   $lblCopy.Location = New-Object Drawing.Point(67, 37)
   $lblCopy.Size = New-Object Drawing.Size(360, 23)
   $lblCopy.Text = "(C) 2011-2013 greg zakharov gregzakh@gmail.com"
   #
   #
   $btnExit.Location = New-Object Drawing.Point(135, 67)
   $btnExit.Text = "OK"
   #
   #
   $frmInfo.AcceptButton = $btnExit
   $frmInfo.CancelButton = $btnExit
   $frmInfo.ClientSize = New-Object Drawing.Size(350, 110)
   $frmInfo.ControlBox = $false
   $frmInfo.Controls.AddRange(@($pbImage, $lblName, $lblCopy, $btnExit))
   $frmInfo.ShowInTaskBar = $false
   $frmInfo.StartPosition = "CenterScreen"
   $frmInfo.Text = "About..."
   $frmInfo.Add_Load({$pbImage.Image = $ico.ToBitmap()})
 
   [void]$frmInfo.ShowDialog()
 }
 
 ##################################################################################################
 
 $i_1 = "/9j/4AAQSkZJRgABAQEAYABgAAD/2wBDAAgGBgcGBQgHBwcJCQgKDBQNDAsLDBkSEw8UHRofHh0aHBwgJC4nI" + `
        "CIsIxwcKDcpLDAxNDQ0Hyc5PTgyPC4zNDL/2wBDAQkJCQwLDBgNDRgyIRwhMjIyMjIyMjIyMjIyMjIyMjIyMj" + `
        "IyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjL/wAARCAARABYDASIAAhEBAxEB/8QAHwAAAQUBAQEBAQE" + `
        "AAAAAAAAAAAECAwQFBgcICQoL/8QAtRAAAgEDAwIEAwUFBAQAAAF9AQIDAAQRBRIhMUEGE1FhByJxFDKBkaEI" + `
        "I0KxwRVS0fAkM2JyggkKFhcYGRolJicoKSo0NTY3ODk6Q0RFRkdISUpTVFVWV1hZWmNkZWZnaGlqc3R1dnd4e" + `
        "XqDhIWGh4iJipKTlJWWl5iZmqKjpKWmp6ipqrKztLW2t7i5usLDxMXGx8jJytLT1NXW19jZ2uHi4+Tl5ufo6e" + `
        "rx8vP09fb3+Pn6/8QAHwEAAwEBAQEBAQEBAQAAAAAAAAECAwQFBgcICQoL/8QAtREAAgECBAQDBAcFBAQAAQJ" + `
        "3AAECAxEEBSExBhJBUQdhcRMiMoEIFEKRobHBCSMzUvAVYnLRChYkNOEl8RcYGRomJygpKjU2Nzg5OkNERUZH" + `
        "SElKU1RVVldYWVpjZGVmZ2hpanN0dXZ3eHl6goOEhYaHiImKkpOUlZaXmJmaoqOkpaanqKmqsrO0tba3uLm6w" + `
        "sPExcbHyMnK0tPU1dbX2Nna4uPk5ebn6Onq8vP09fb3+Pn6/9oADAMBAAIRAxEAPwD2vUL4x3f2bzpYdqK4MK" + `
        "B3csWGACD0CknAJ+mDmk+tX9rDIRZyXYQEqWhmjdhjoQIiuffIH07V/EUs+n6xbaiLW4ng2BCIE3sCBIOR/wA" + `
        "DH5H8cu48WTSQSRrpGrZZSB/ox9PrXkVq2JhVlyxbXTTT8jqjCDS1X3nZabqMOp2nnw7lwxSSNxho3HVWHYii" + `
        "sXwUk/2DUbma3ltxd6jNPGky7W2NjGR26UV6lNuUE5KzOaSSbsdLRRRViCiiigD/2Q=="
 
 $i_2 = "/9j/4AAQSkZJRgABAQEAYABgAAD/2wBDAAgGBgcGBQgHBwcJCQgKDBQNDAsLDBkSEw8UHRofHh0aHBwgJC4nI" + `
        "CIsIxwcKDcpLDAxNDQ0Hyc5PTgyPC4zNDL/2wBDAQkJCQwLDBgNDRgyIRwhMjIyMjIyMjIyMjIyMjIyMjIyMj" + `
        "IyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjL/wAARCAASABADASIAAhEBAxEB/8QAHwAAAQUBAQEBAQE" + `
        "AAAAAAAAAAAECAwQFBgcICQoL/8QAtRAAAgEDAwIEAwUFBAQAAAF9AQIDAAQRBRIhMUEGE1FhByJxFDKBkaEI" + `
        "I0KxwRVS0fAkM2JyggkKFhcYGRolJicoKSo0NTY3ODk6Q0RFRkdISUpTVFVWV1hZWmNkZWZnaGlqc3R1dnd4e" + `
        "XqDhIWGh4iJipKTlJWWl5iZmqKjpKWmp6ipqrKztLW2t7i5usLDxMXGx8jJytLT1NXW19jZ2uHi4+Tl5ufo6e" + `
        "rx8vP09fb3+Pn6/8QAHwEAAwEBAQEBAQEBAQAAAAAAAAECAwQFBgcICQoL/8QAtREAAgECBAQDBAcFBAQAAQJ" + `
        "3AAECAxEEBSExBhJBUQdhcRMiMoEIFEKRobHBCSMzUvAVYnLRChYkNOEl8RcYGRomJygpKjU2Nzg5OkNERUZH" + `
        "SElKU1RVVldYWVpjZGVmZ2hpanN0dXZ3eHl6goOEhYaHiImKkpOUlZaXmJmaoqOkpaanqKmqsrO0tba3uLm6w" + `
        "sPExcbHyMnK0tPU1dbX2Nna4uPk5ebn6Onq8vP09fb3+Pn6/9oADAMBAAIRAxEAPwD2gSWscJmuppVzLIMiV8" + `
        "cM3YHgACq8WoxpfbIpfMtZHCgksxVuBjkf3uOvcdOaydZgvr6zRLBAxMlyjls4H7zHb8aybXTdSPiTTpLmOSO" + `
        "J3WNsoeTH86ln/jYhOTgZ7Dito04OHM5a9jmnWqKooRjdaanayaNpcsjSSabZu7kszNApJJ6knFOg0nTraZZo" + `
        "NPtYpV+68cKqw7dQKKKxOk//2Q=="
 
 $i_3 = "/9j/4AAQSkZJRgABAQEAYABgAAD/4QAWRXhpZgAASUkqAAgAAAAAAAAAAAD/2wBDAAgGBgcGBQgHBwcJCQgKD" + `
        "BQNDAsLDBkSEw8UHRofHh0aHBwgJC4nICIsIxwcKDcpLDAxNDQ0Hyc5PTgyPC4zNDL/2wBDAQkJCQwLDBgNDR" + `
        "gyIRwhMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjL/wAARCAANAAg" + `
        "DASIAAhEBAxEB/8QAHwAAAQUBAQEBAQEAAAAAAAAAAAECAwQFBgcICQoL/8QAtRAAAgEDAwIEAwUFBAQAAAF9" + `
        "AQIDAAQRBRIhMUEGE1FhByJxFDKBkaEII0KxwRVS0fAkM2JyggkKFhcYGRolJicoKSo0NTY3ODk6Q0RFRkdIS" + `
        "UpTVFVWV1hZWmNkZWZnaGlqc3R1dnd4eXqDhIWGh4iJipKTlJWWl5iZmqKjpKWmp6ipqrKztLW2t7i5usLDxM" + `
        "XGx8jJytLT1NXW19jZ2uHi4+Tl5ufo6erx8vP09fb3+Pn6/8QAHwEAAwEBAQEBAQEBAQAAAAAAAAECAwQFBgc" + `
        "ICQoL/8QAtREAAgECBAQDBAcFBAQAAQJ3AAECAxEEBSExBhJBUQdhcRMiMoEIFEKRobHBCSMzUvAVYnLRChYk" + `
        "NOEl8RcYGRomJygpKjU2Nzg5OkNERUZHSElKU1RVVldYWVpjZGVmZ2hpanN0dXZ3eHl6goOEhYaHiImKkpOUl" + `
        "ZaXmJmaoqOkpaanqKmqsrO0tba3uLm6wsPExcbHyMnK0tPU1dbX2Nna4uPk5ebn6Onq8vP09fb3+Pn6/9oADA" + `
        "MBAAIRAxEAPwDtvDfjq4XxvqXhzUBLPG9/OlpKql2jw7fI2OdmBwf4e/y/dK63RPC1jol/qOoRjzb6/nklkmY" + `
        "cqrOWCL6AcZ9SMnsAVlRjOMbTZ6GZVsNVrKWHjZWV/N9Xbp/TP//Z"
 
 ##################################################################################################
 
 frmMain_Show
`

