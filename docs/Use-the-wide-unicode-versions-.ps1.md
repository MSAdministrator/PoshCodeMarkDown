---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 401
Published Date: 
Archived Date: 2016-06-19t00
---

#  - 

## Description



## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 param(
    [string] $dirRoot = ".",
    [string] $Spec = "*.*"
    )
 
 #
 #
 #
 #
 #
 #
 
 $provider = new-object Microsoft.VisualBasic.VBCodeProvider
 $params = new-object System.CodeDom.Compiler.CompilerParameters
 $params.GenerateInMemory = $True
 $refs = "System.dll","Microsoft.VisualBasic.dll"
 $params.ReferencedAssemblies.AddRange($refs)
 
 $txtCode = @'
 Imports System
 Imports System.Runtime.InteropServices
 Class FindFiles
 
 Const ERROR_SUCCESS As Long = 0
 Private Const MAX_PREFERRED_LENGTH As Long = -1
 
 <System.Runtime.InteropServices.StructLayoutAttribute(System.Runtime.InteropServices.LayoutKind.Sequential, CharSet:=System.Runtime.InteropServices.CharSet.[Unicode])>  _
 Public Structure WIN32_FIND_DATAW
     '''DWORD->unsigned int
     Public dwFileAttributes As UInteger
     '''FILETIME->_FILETIME
     Public ftCreationTime As FILETIME
     '''FILETIME->_FILETIME
     Public ftLastAccessTime As FILETIME
     '''FILETIME->_FILETIME
     Public ftLastWriteTime As FILETIME
     '''DWORD->unsigned int
     Public nFileSizeHigh As UInteger
     '''DWORD->unsigned int
     Public nFileSizeLow As UInteger
     '''DWORD->unsigned int
     Public dwReserved0 As UInteger
     '''DWORD->unsigned int
     Public dwReserved1 As UInteger
     '''WCHAR[260]
     <System.Runtime.InteropServices.MarshalAsAttribute(System.Runtime.InteropServices.UnmanagedType.ByValTStr, SizeConst:=260)>  _
     Public cFileName As String
     '''WCHAR[14]
     <System.Runtime.InteropServices.MarshalAsAttribute(System.Runtime.InteropServices.UnmanagedType.ByValTStr, SizeConst:=14)>  _
     Public cAlternateFileName As String
 End Structure
 
 <System.Runtime.InteropServices.StructLayoutAttribute(System.Runtime.InteropServices.LayoutKind.Sequential)>  _
 Public Structure FILETIME
     '''DWORD->unsigned int
     Public dwLowDateTime As UInteger
     '''DWORD->unsigned int
     Public dwHighDateTime As UInteger
 End Structure
 
 Partial Public Class NativeMethods
     
     '''Return Type: HANDLE->void*
     '''lpFileName: LPCWSTR->WCHAR*
     '''lpFindFileData: LPWIN32_FIND_DATAW->_WIN32_FIND_DATAW*
     <System.Runtime.InteropServices.DllImportAttribute("kernel32.dll", EntryPoint:="FindFirstFileW")>  _
     Public Shared Function FindFirstFileW(<System.Runtime.InteropServices.InAttribute(), System.Runtime.InteropServices.MarshalAsAttribute(System.Runtime.InteropServices.UnmanagedType.LPWStr)> ByVal lpFileName As String, <System.Runtime.InteropServices.OutAttribute()> ByRef lpFindFileData As WIN32_FIND_DATAW) As System.IntPtr
     End Function
    
     '''Return Type: BOOL->int
     '''hFindFile: HANDLE->void*
     '''lpFindFileData: LPWIN32_FIND_DATAW->_WIN32_FIND_DATAW*
     <System.Runtime.InteropServices.DllImportAttribute("kernel32.dll", EntryPoint:="FindNextFileW")>  _
     Public Shared Function FindNextFileW(<System.Runtime.InteropServices.InAttribute()> ByVal hFindFile As System.IntPtr, <System.Runtime.InteropServices.OutAttribute()> ByRef lpFindFileData As WIN32_FIND_DATAW) As <System.Runtime.InteropServices.MarshalAsAttribute(System.Runtime.InteropServices.UnmanagedType.Bool)> Boolean
     End Function
 
     '''Return Type: BOOL->int
     '''hFindFile: HANDLE->void*
     <System.Runtime.InteropServices.DllImportAttribute("kernel32.dll", EntryPoint:="FindClose")>  _
     Public Shared Function FindClose(ByVal hFindFile As System.IntPtr) As <System.Runtime.InteropServices.MarshalAsAttribute(System.Runtime.InteropServices.UnmanagedType.Bool)> Boolean
     End Function
 End Class
 
 
 Private Const FILE_ATTRIBUTE_DIRECTORY As Long = &H10
     Dim FFW as New NativeMethods
 
 Function Main(ByVal dirRoot As String, ByVal sFileSpec As String) As Long
     Dim result As Long
 
     result = FindFiles(dirRoot, sFileSpec) 
 
     main = result										' Return the result
 End Function
 
 Function FindFiles(ByRef sDir As String, ByVal sFileSpec as String)
     Dim FindFileData as WIN32_FIND_DATAW
     Dim hFile As Long
     Dim sFullPath As String
 
     sFullPath = "\\?\" & sDir
 
     'console.writeline(sFullPath & "\" & sFileSpec)
 
     hFile = FFW.FindFirstFileW(sFullPath & "\" & sFileSpec, FindFileData)					' Find the first object
     if hFile > 0 Then												' Has something been found?
       If (FindFileData.dwFileAttributes AND FILE_ATTRIBUTE_DIRECTORY)  <> FILE_ATTRIBUTE_DIRECTORY Then		' Is this a file?
         console.writeline(sFullPath & "\" & FindFileData.cFileName)						' Yes, report the full path and filename
       End If
 
       While FFW.FindNextFileW(hFile, FindFileData)								' For all the items in this directory
         If (FindFileData.dwFileAttributes AND FILE_ATTRIBUTE_DIRECTORY) <> FILE_ATTRIBUTE_DIRECTORY Then	' Is this a file?
           console.writeline(sFullPath & "\" & FindFileData.cFileName)						' Yes, report the full path and filename
         End If
       End While
       FFW.FindClose(hFile)											' Close the handle
       FindFileData = Nothing
     End If
 
     hFile = FFW.FindFirstFileW(sFullPath & "\" & "*.*", FindFileData)						' Repeat the process looking for sub-directories using *.*
     if hFile > 0 Then
       If (FindFileData.dwFileAttributes AND FILE_ATTRIBUTE_DIRECTORY) AND _
           (FindFileData.cFileName <> ".") AND (FindFileData.cFileName <> "..") Then
         Call FindFiles(sDir & "\" & FindFileData.cFileName, sFileSpec)					' Recurse
       End If
 
       While FFW.FindNextFileW(hFile, FindFileData)
         If (FindFileData.dwFileAttributes AND FILE_ATTRIBUTE_DIRECTORY) AND _
             (FindFileData.cFileName <> ".") AND (FindFileData.cFileName <> "..") Then
           Call FindFiles(sDir & "\" & FindFileData.cFileName, sFileSpec)					' Recurse
         End If
       End While
       FFW.FindClose(hFile)											' Close the handle
       FindFileData = Nothing
     End If
 
 End Function
 
 end class
 
 '@
 
 
 $cr = $provider.CompileAssemblyFromSource($params, $txtCode)
 if ($cr.Errors.Count) {
     $codeLines = $txtCode.Split("`n");
     foreach ($ce in $cr.Errors)
     {
         write-host "Error: $($codeLines[$($ce.Line - 1)])"
         write-host $ce
         #$ce out-default
     }
     Throw "INVALID DATA: Errors encountered while compiling code"
  }
 $mAssembly = $cr.CompiledAssembly
 $instance = $mAssembly.CreateInstance("FindFiles")
 
 $result = $instance.main($dirRoot, $Spec)
`

