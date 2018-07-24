---
Author: foobar
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 913
Published Date: 
Archived Date: 2009-03-11t04
---

# tabexpansion for v2ctp3 - 

## Description

ported tabexpansion from v2ctp2 to v2ctp3 and extended. please dot souce this script file to use.

## Comments



## Usage



## TODO



## script

`write-classnames`

## Code

`#
 #
 #################
 ##
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 if ( Test-Path $PSHOME\ProgIDs.txt )
 {
 $_ProgID = type $PSHOME\ProgIDs.txt -ReadCount 0
 }
 else
 {
 $_HKCR = [Microsoft.Win32.Registry]::ClassesRoot.OpenSubKey("CLSID\")
 $_ProgID = New-Object ( [System.Collections.Generic.List``1].MakeGenericType([String]) )
 foreach ( $_subkey in $_HKCR.GetSubKeyNames() )
 {
 foreach ( $_i in [Microsoft.Win32.Registry]::ClassesRoot.OpenSubKey("CLSID\$_subkey\ProgID") )
 {
 if ($_i -ne $null)
 {
 $_ProgID.Add($_i.GetValue(""))
 }
 }
 }
 '$_ProgID was updated...' | Out-Host
 $_ProgID = $_ProgID|sort -Unique
 
 Set-Content -Value $_ProgID -Path $PSHOME\ProgIDs.txt -Verbose
 }
 
 
 if ( Test-Path $PSHOME\TypeNames.txt )
 {
 $_TypeNames = type $PSHOME\TypeNames.txt -ReadCount 0
 }
 else
 {
 $_TypeNames = New-Object ( [System.Collections.Generic.List``1].MakeGenericType([String]) )
 foreach ( $_asm in [AppDomain]::CurrentDomain.GetAssemblies() )
 {
 foreach ( $_type in $_asm.GetTypes() )
 {
 $_TypeNames.Add($_type.FullName)
 }
 }
 '$_TypeNames was updated...' | Out-Host
 $_TypeNames = $_TypeNames | sort -Unique
 
 Set-Content -Value $_TypeNames -Path $PSHOME\TypeNames.txt -Verbose
 }
 
 if ( Test-Path $PSHOME\TypeNames_System.txt )
 {
 $_TypeNames_System = type $PSHOME\TypeNames_System.txt -ReadCount 0
 }
 else
 {
 $_TypeNames_System = $_TypeNames -like "System.*" -replace '^System\.'
 '$_TypeNames_System was updated...' | Out-Host
 Set-Content -Value $_TypeNames_System -Path $PSHOME\TypeNames_System.txt -Verbose
 }
 
 if ( Test-Path $PSHOME\WMIClasses.txt )
 {
 $_WMIClasses = type $PSHOME\WMIClasses.txt -ReadCount 0
 }
 else
 {
 $_WMIClasses = New-Object ( [System.Collections.Generic.List``1].MakeGenericType([String]) )
 foreach ( $_class in gwmi -List )
 {
 $_WMIClasses.Add($_class.Name)
 }
 $_WMIClasses = $_WMIClasses | sort -Unique
 '$_WMIClasses was updated...' | Out-Host
 Set-Content -Value $_WMIClasses -Path $PSHOME\WMIClasses.txt -Verbose
 }
 
 [Reflection.Assembly]::LoadWithPartialName( "System.Windows.Forms" ) | Out-Null
 $global:_cmdstack = New-Object Collections.Stack
 $global:_snapin = $null
 $global:_TypeAccelerators = [type]::gettype("System.Management.Automation.TypeAccelerators")::get.keys | sort
 
 iex (@'
 function prompt {
 if ($_cmdstack.Count -gt 0) {
 $line = $global:_cmdstack.Pop() -replace '([[\]\(\)+{}?~%])','{$1}'
 [System.Windows.Forms.SendKeys]::SendWait($line)
 }
 '@ + @"
 ${function:prompt}
 }
 "@)
 
 function Write-ClassNames ( $data, $i, $prefix='', $sep='.' )
 {
 $preItem = ""
 foreach ( $class in $data -like $_opt )
 {
 $Item = $class.Split($sep)
 if ( $preItem -ne $Item[$i] )
 {
 if ( $i+1 -eq $Item.Count )
 {
 if ( $prefix -eq "[" )
 {
 $suffix = "]"
 }
 elseif ( $sep -eq "_" )
 {
 $suffix = ""
 }
 else
 {
 $suffix = " "
 }
 }
 else
 {
 $suffix = ""
 }
 $prefix + $_opt.Substring(0, $_opt.LastIndexOf($sep)+1) + $Item[$i] + $suffix
 
 $preItem = $Item[$i]
 }
 }
 }
 
 function Get-PipeLineObject {
 
 $i = -2
 $property = $null
 do {
 $str = $line.Split("|")
 $_cmdlet = [regex]::Split($str[$i], '[|;=]')[-1]
 
 $_cmdlet = $_cmdlet.Trim().Split()[0]
 
 if ( $_cmdlet -eq "?" )
 {
 $_cmdlet = "Where-Object"
 }
 
 $global:_exp = $_cmdlet
 
 $_cmdlet = @(Get-Command -type 'cmdlet,alias' $_cmdlet)[0]
 
 while ($_cmdlet.CommandType -eq 'alias')
 {
 $_cmdlet = @(Get-Command -type 'cmdlet,alias' $_cmdlet.Definition)[0]
 }
 
 if ( "Select-Object" -eq $_cmdlet )
 {
 if ( $str[$i] -match '\s+-Exp\w*[\s:]+(\w+)' )
 {
 $property = $Matches[1] + ";" + $property
 }
 }
 
 $i--
 } while ( "Get-Unique", "Select-Object", "Sort-Object", "Tee-Object", "Where-Object" -contains $_cmdlet )
 
 if ( $global:_forgci -eq $null )
 {
 $a = @(ls "Alias:\")[0]
 $e = @(ls "Env:\")[0]
 $f = @(ls "Function:\")[0]
 $h = @(ls "HKCU:\")[0]
 $v = @(ls "Variable:\")[0]
 $c = @(ls "cert:\")[0]
 $global:_forgci = gi $PSHOME\powershell.exe |
 Add-Member  'NoteProperty'  CommandType  $f.CommandType -P |
 Add-Member  'NoteProperty'  Definition  $a.Definition -P |
 Add-Member  'NoteProperty'  Description  $a.Description -P |
 Add-Member  'NoteProperty'  Key  $e.Key -P |
 Add-Member  'NoteProperty'  Location  $c.Location -P |
 Add-Member  'NoteProperty'  LocationName  $c.LocationName -P |
 Add-Member  'NoteProperty'  Options  $a.Options -P |
 Add-Member  'NoteProperty'  ReferencedCommand  $a.ReferencedCommand -P |
 Add-Member  'NoteProperty'  ResolvedCommand  $a.ResolvedCommand -P |
 Add-Member  'NoteProperty'  ScriptBlock  $f.ScriptBlock -P |
 Add-Member  'NoteProperty'  StoreNames  $c.StoreNames -P |
 Add-Member  'NoteProperty'  SubKeyCount  $h.SubKeyCount -P |
 Add-Member  'NoteProperty'  Value  $e.Value -P |
 Add-Member  'NoteProperty'  ValueCount  $h.ValueCount -P |
 Add-Member  'NoteProperty'  Visibility  $a.Visibility -P |
 Add-Member  'NoteProperty'  Property  $h.Property -P |
 Add-Member  'NoteProperty'  ResolvedCommandName  $a.ResolvedCommandName -P |
 Add-Member  'ScriptMethod'  Close  {} -P |
 Add-Member  'ScriptMethod'  CreateSubKey  {} -P |
 Add-Member  'ScriptMethod'  DeleteSubKey  {} -P |
 Add-Member  'ScriptMethod'  DeleteSubKeyTree  {} -P |
 Add-Member  'ScriptMethod'  DeleteValue  {} -P |
 Add-Member  'ScriptMethod'  Flush  {} -P |
 Add-Member  'ScriptMethod'  GetSubKeyNames  {} -P |
 Add-Member  'ScriptMethod'  GetValue  {} -P |
 Add-Member  'ScriptMethod'  GetValueKind  {} -P |
 Add-Member  'ScriptMethod'  GetValueNames  {} -P |
 Add-Member  'ScriptMethod'  IsValidValue  {} -P |
 Add-Member  'ScriptMethod'  OpenSubKey  {} -P |
 Add-Member  'ScriptMethod'  SetValue  {} -P
 }
 
 if ( $global:_mix -eq $null )
 {
 $f = gi $PSHOME\powershell.exe
 $t = [type]
 $s = ""
 $global:_mix = `
 Add-Member -InputObject (New-Object PSObject)  'NoteProperty'  Mode  $f.Mode -P |
 Add-Member  'NoteProperty'  Assembly  $t.Assembly -P |
 Add-Member  'NoteProperty'  AssemblyQualifiedName  $t.AssemblyQualifiedName -P |
 Add-Member  'NoteProperty'  Attributes  $f.Attributes -P |
 Add-Member  'NoteProperty'  BaseType  $t.BaseType -P |
 Add-Member  'NoteProperty'  ContainsGenericParameters  $t.ContainsGenericParameters -P |
 Add-Member  'NoteProperty'  CreationTime  $f.CreationTime -P |
 Add-Member  'NoteProperty'  CreationTimeUtc  $f.CreationTimeUtc -P |
 Add-Member  'NoteProperty'  DeclaringMethod  $t.DeclaringMethod -P |
 Add-Member  'NoteProperty'  DeclaringType  $t.DeclaringType -P |
 Add-Member  'NoteProperty'  Exists  $f.Exists -P |
 Add-Member  'NoteProperty'  Extension  $f.Extension -P |
 Add-Member  'NoteProperty'  FullName  $f.FullName -P |
 Add-Member  'NoteProperty'  GenericParameterAttributes  $t.GenericParameterAttributes -P |
 Add-Member  'NoteProperty'  GenericParameterPosition  $t.GenericParameterPosition -P |
 Add-Member  'NoteProperty'  GUID  $t.GUID -P |
 Add-Member  'NoteProperty'  HasElementType  $t.HasElementType -P |
 Add-Member  'NoteProperty'  IsAbstract  $t.IsAbstract -P |
 Add-Member  'NoteProperty'  IsAnsiClass  $t.IsAnsiClass -P |
 Add-Member  'NoteProperty'  IsArray  $t.IsArray -P |
 Add-Member  'NoteProperty'  IsAutoClass  $t.IsAutoClass -P |
 Add-Member  'NoteProperty'  IsAutoLayout  $t.IsAutoLayout -P |
 Add-Member  'NoteProperty'  IsByRef  $t.IsByRef -P |
 Add-Member  'NoteProperty'  IsClass  $t.IsClass -P |
 Add-Member  'NoteProperty'  IsCOMObject  $t.IsCOMObject -P |
 Add-Member  'NoteProperty'  IsContextful  $t.IsContextful -P |
 Add-Member  'NoteProperty'  IsEnum  $t.IsEnum -P |
 Add-Member  'NoteProperty'  IsExplicitLayout  $t.IsExplicitLayout -P |
 Add-Member  'NoteProperty'  IsGenericParameter  $t.IsGenericParameter -P |
 Add-Member  'NoteProperty'  IsGenericType  $t.IsGenericType -P |
 Add-Member  'NoteProperty'  IsGenericTypeDefinition  $t.IsGenericTypeDefinition -P |
 Add-Member  'NoteProperty'  IsImport  $t.IsImport -P |
 Add-Member  'NoteProperty'  IsInterface  $t.IsInterface -P |
 Add-Member  'NoteProperty'  IsLayoutSequential  $t.IsLayoutSequential -P |
 Add-Member  'NoteProperty'  IsMarshalByRef  $t.IsMarshalByRef -P |
 Add-Member  'NoteProperty'  IsNested  $t.IsNested -P |
 Add-Member  'NoteProperty'  IsNestedAssembly  $t.IsNestedAssembly -P |
 Add-Member  'NoteProperty'  IsNestedFamANDAssem  $t.IsNestedFamANDAssem -P |
 Add-Member  'NoteProperty'  IsNestedFamily  $t.IsNestedFamily -P |
 Add-Member  'NoteProperty'  IsNestedFamORAssem  $t.IsNestedFamORAssem -P |
 Add-Member  'NoteProperty'  IsNestedPrivate  $t.IsNestedPrivate -P |
 Add-Member  'NoteProperty'  IsNestedPublic  $t.IsNestedPublic -P |
 Add-Member  'NoteProperty'  IsNotPublic  $t.IsNotPublic -P |
 Add-Member  'NoteProperty'  IsPointer  $t.IsPointer -P |
 Add-Member  'NoteProperty'  IsPrimitive  $t.IsPrimitive -P |
 Add-Member  'NoteProperty'  IsPublic  $t.IsPublic -P |
 Add-Member  'NoteProperty'  IsSealed  $t.IsSealed -P |
 Add-Member  'NoteProperty'  IsSerializable  $t.IsSerializable -P |
 Add-Member  'NoteProperty'  IsSpecialName  $t.IsSpecialName -P |
 Add-Member  'NoteProperty'  IsUnicodeClass  $t.IsUnicodeClass -P |
 Add-Member  'NoteProperty'  IsValueType  $t.IsValueType -P |
 Add-Member  'NoteProperty'  IsVisible  $t.IsVisible -P |
 Add-Member  'NoteProperty'  LastAccessTime  $f.LastAccessTime -P |
 Add-Member  'NoteProperty'  LastAccessTimeUtc  $f.LastAccessTimeUtc -P |
 Add-Member  'NoteProperty'  LastWriteTime  $f.LastWriteTime -P |
 Add-Member  'NoteProperty'  LastWriteTimeUtc  $f.LastWriteTimeUtc -P |
 Add-Member  'NoteProperty'  MemberType  $t.MemberType -P |
 Add-Member  'NoteProperty'  MetadataToken  $t.MetadataToken -P |
 Add-Member  'NoteProperty'  Module  $t.Module -P |
 Add-Member  'NoteProperty'  Name  $t.Name -P |
 Add-Member  'NoteProperty'  Namespace  $t.Namespace -P |
 Add-Member  'NoteProperty'  Parent  $f.Parent -P |
 Add-Member  'NoteProperty'  ReflectedType  $t.ReflectedType -P |
 Add-Member  'NoteProperty'  Root  $f.Root -P |
 Add-Member  'NoteProperty'  StructLayoutAttribute  $t.StructLayoutAttribute -P |
 Add-Member  'NoteProperty'  TypeHandle  $t.TypeHandle -P |
 Add-Member  'NoteProperty'  TypeInitializer  $t.TypeInitializer -P |
 Add-Member  'NoteProperty'  UnderlyingSystemType  $t.UnderlyingSystemType -P |
 Add-Member  'NoteProperty'  PSChildName  $f.PSChildName -P |
 Add-Member  'NoteProperty'  PSDrive  $f.PSDrive -P |
 Add-Member  'NoteProperty'  PSIsContainer  $f.PSIsContainer -P |
 Add-Member  'NoteProperty'  PSParentPath  $f.PSParentPath -P |
 Add-Member  'NoteProperty'  PSPath  $f.PSPath -P |
 Add-Member  'NoteProperty'  PSProvider  $f.PSProvider -P |
 Add-Member  'NoteProperty'  BaseName  $f.BaseName -P |
 Add-Member  'ScriptMethod'  Clone  {} -P |
 Add-Member  'ScriptMethod'  CompareTo  {} -P |
 Add-Member  'ScriptMethod'  Contains  {} -P |
 Add-Member  'ScriptMethod'  CopyTo  {} -P |
 Add-Member  'ScriptMethod'  Create  {} -P |
 Add-Member  'ScriptMethod'  CreateObjRef  {} -P |
 Add-Member  'ScriptMethod'  CreateSubdirectory  {} -P |
 Add-Member  'ScriptMethod'  Delete  {} -P |
 Add-Member  'ScriptMethod'  EndsWith  {} -P |
 Add-Member  'ScriptMethod'  FindInterfaces  {} -P |
 Add-Member  'ScriptMethod'  FindMembers  {} -P |
 Add-Member  'ScriptMethod'  GetAccessControl  {} -P |
 Add-Member  'ScriptMethod'  GetArrayRank  {} -P |
 Add-Member  'ScriptMethod'  GetConstructor  {} -P |
 Add-Member  'ScriptMethod'  GetConstructors  {} -P |
 Add-Member  'ScriptMethod'  GetCustomAttributes  {} -P |
 Add-Member  'ScriptMethod'  GetDefaultMembers  {} -P |
 Add-Member  'ScriptMethod'  GetDirectories  {} -P |
 Add-Member  'ScriptMethod'  GetElementType  {} -P |
 Add-Member  'ScriptMethod'  GetEnumerator  {} -P |
 Add-Member  'ScriptMethod'  GetEvent  {} -P |
 Add-Member  'ScriptMethod'  GetEvents  {} -P |
 Add-Member  'ScriptMethod'  GetField  {} -P |
 Add-Member  'ScriptMethod'  GetFields  {} -P |
 Add-Member  'ScriptMethod'  GetFiles  {} -P |
 Add-Member  'ScriptMethod'  GetFileSystemInfos  {} -P |
 Add-Member  'ScriptMethod'  GetGenericArguments  {} -P |
 Add-Member  'ScriptMethod'  GetGenericParameterConstraints  {} -P |
 Add-Member  'ScriptMethod'  GetGenericTypeDefinition  {} -P |
 Add-Member  'ScriptMethod'  GetInterface  {} -P |
 Add-Member  'ScriptMethod'  GetInterfaceMap  {} -P |
 Add-Member  'ScriptMethod'  GetInterfaces  {} -P |
 Add-Member  'ScriptMethod'  GetLifetimeService  {} -P |
 Add-Member  'ScriptMethod'  GetMember  {} -P |
 Add-Member  'ScriptMethod'  GetMembers  {} -P |
 Add-Member  'ScriptMethod'  GetMethod  {} -P |
 Add-Member  'ScriptMethod'  GetMethods  {} -P |
 Add-Member  'ScriptMethod'  GetNestedType  {} -P |
 Add-Member  'ScriptMethod'  GetNestedTypes  {} -P |
 Add-Member  'ScriptMethod'  GetObjectData  {} -P |
 Add-Member  'ScriptMethod'  GetProperties  {} -P |
 Add-Member  'ScriptMethod'  GetProperty  {} -P |
 Add-Member  'ScriptMethod'  GetTypeCode  {} -P |
 Add-Member  'ScriptMethod'  IndexOf  {} -P |
 Add-Member  'ScriptMethod'  IndexOfAny  {} -P |
 Add-Member  'ScriptMethod'  InitializeLifetimeService  {} -P |
 Add-Member  'ScriptMethod'  Insert  {} -P |
 Add-Member  'ScriptMethod'  InvokeMember  {} -P |
 Add-Member  'ScriptMethod'  IsAssignableFrom  {} -P |
 Add-Member  'ScriptMethod'  IsDefined  {} -P |
 Add-Member  'ScriptMethod'  IsInstanceOfType  {} -P |
 Add-Member  'ScriptMethod'  IsNormalized  {} -P |
 Add-Member  'ScriptMethod'  IsSubclassOf  {} -P |
 Add-Member  'ScriptMethod'  LastIndexOf  {} -P |
 Add-Member  'ScriptMethod'  LastIndexOfAny  {} -P |
 Add-Member  'ScriptMethod'  MakeArrayType  {} -P |
 Add-Member  'ScriptMethod'  MakeByRefType  {} -P |
 Add-Member  'ScriptMethod'  MakeGenericType  {} -P |
 Add-Member  'ScriptMethod'  MakePointerType  {} -P |
 Add-Member  'ScriptMethod'  MoveTo  {} -P |
 Add-Member  'ScriptMethod'  Normalize  {} -P |
 Add-Member  'ScriptMethod'  PadLeft  {} -P |
 Add-Member  'ScriptMethod'  PadRight  {} -P |
 Add-Member  'ScriptMethod'  Refresh  {} -P |
 Add-Member  'ScriptMethod'  Remove  {} -P |
 Add-Member  'ScriptMethod'  Replace  {} -P |
 Add-Member  'ScriptMethod'  SetAccessControl  {} -P |
 Add-Member  'ScriptMethod'  Split  {} -P |
 Add-Member  'ScriptMethod'  StartsWith  {} -P |
 Add-Member  'ScriptMethod'  Substring  {} -P |
 Add-Member  'ScriptMethod'  ToCharArray  {} -P |
 Add-Member  'ScriptMethod'  ToLower  {} -P |
 Add-Member  'ScriptMethod'  ToLowerInvariant  {} -P |
 Add-Member  'ScriptMethod'  ToUpper  {} -P |
 Add-Member  'ScriptMethod'  ToUpperInvariant  {} -P |
 Add-Member  'ScriptMethod'  Trim  {} -P |
 Add-Member  'ScriptMethod'  TrimEnd  {} -P |
 Add-Member  'ScriptMethod'  TrimStart  {} -P |
 Add-Member  'NoteProperty'  Chars  $s.Chars -P
 }
 
 
 if ( "Add-Member" -eq $_cmdlet )
 {
 $global:_dummy = $null
 }
 
 
 if ( "Compare-Object" -eq $_cmdlet )
 {
 $global:_dummy =  (Compare-Object 1 2)[0]
 }
 
 
 if ( "ConvertFrom-SecureString" -eq $_cmdlet )
 {
 $global:_dummy = $null
 }
 
 
 if ( "ConvertTo-SecureString" -eq $_cmdlet )
 {
 $global:_dummy = convertto-securestring "P@ssW0rD!" -asplaintext -force
 }
 
 
 if ( "ForEach-Object" -eq $_cmdlet )
 {
 $global:_dummy = $null
 }
 
 
 if ( "Get-Acl" -eq $_cmdlet )
 {
 $global:_dummy = Get-Acl
 }
 
 
 if ( "Get-Alias" -eq $_cmdlet )
 {
 $global:_dummy = (Get-Alias)[0]
 }
 
 
 if ( "Get-AuthenticodeSignature" -eq $_cmdlet )
 {
 $global:_dummy = Get-AuthenticodeSignature $PSHOME\powershell.exe
 }
 
 
 if ( "Get-ChildItem" -eq $_cmdlet )
 {
 $global:_dummy = $global:_forgci
 }
 
 
 if ( "Get-Command" -eq $_cmdlet )
 {
 $global:_dummy = @(iex $str[$i+1])[0]
 }
 
 
 if ( "Get-Content" -eq $_cmdlet )
 {
 $global:_dummy = (type $PSHOME\profile.ps1)[0]
 }
 
 
 if ( "Get-Credential" -eq $_cmdlet )
 {
 $global:_dummy = $null
 }
 
 
 if ( "Get-Culture" -eq $_cmdlet )
 {
 $global:_dummy = Get-Culture
 }
 
 
 if ( "Get-Date" -eq $_cmdlet )
 {
 $global:_dummy = Get-Date
 }
 
 
 if ( "Get-Event" -eq $_cmdlet )
 {
 $global:_dummy = (Get-Event)[0]
 }
 
 
 if ( "Get-EventLog" -eq $_cmdlet )
 {
 $global:_dummy = @(iex $str[$i+1])[0]
 }
 
 
 if ( "Get-ExecutionPolicy" -eq $_cmdlet )
 {
 $global:_dummy = Get-ExecutionPolicy
 }
 
 
 if ( "Get-Help" -eq $_cmdlet )
 {
 $global:_dummy = Get-Help Add-Content
 }
 
 
 if ( "Get-History" -eq $_cmdlet )
 {
 $global:_dummy = Get-History -Count 1
 }
 
 
 if ( "Get-Host" -eq $_cmdlet )
 {
 $global:_dummy = Get-Host
 }
 
 
 if ( "Get-Item" -eq $_cmdlet )
 {
 $global:_dummy = $global:_forgci
 }
 
 
 if ( "Get-ItemProperty" -eq $_cmdlet )
 {
 $global:_dummy = $null
 }
 
 
 if ( "Get-Location" -eq $_cmdlet )
 {
 $global:_dummy = Get-Location
 }
 
 
 if ( "Get-Member" -eq $_cmdlet )
 {
 $global:_dummy = (1|Get-Member)[0]
 }
 
 
 if ( "Get-Module" -eq $_cmdlet )
 {
 $global:_dummy = (Get-Module)[0]
 }
 
 
 if ( "Get-PfxCertificate" -eq $_cmdlet )
 {
 $global:_dummy = $null
 }
 
 
 if ( "Get-Process" -eq $_cmdlet )
 {
 $global:_dummy = ps powershell
 }
 
 
 if ( "Get-PSBreakpoint" -eq $_cmdlet )
 {
 $global:_dummy =
 Add-Member -InputObject (New-Object PSObject)  'NoteProperty'  Action  '' -P |
 Add-Member  'NoteProperty'  Command  '' -P |
 Add-Member  'NoteProperty'  Enabled  '' -P |
 Add-Member  'NoteProperty'  HitCount  '' -P |
 Add-Member  'NoteProperty'  Id  '' -P |
 Add-Member  'NoteProperty'  Script  '' -P
 }
 
 
 if ( "Get-PSCallStack" -eq $_cmdlet )
 {
 $global:_dummy = Get-PSCallStack
 }
 
 
 if ( "Get-PSDrive" -eq $_cmdlet )
 {
 $global:_dummy = Get-PSDrive Function
 }
 
 
 if ( "Get-PSProvider" -eq $_cmdlet )
 {
 $global:_dummy = Get-PSProvider FileSystem
 }
 
 
 if ( "Get-PSSnapin" -eq $_cmdlet )
 {
 $global:_dummy = Get-PSSnapin Microsoft.PowerShell.Core
 }
 
 
 if ( "Get-Service" -eq $_cmdlet )
 {
 $global:_dummy = (Get-Service)[0]
 }
 
 
 if ( "Get-TraceSource" -eq $_cmdlet )
 {
 $global:_dummy = Get-TraceSource AddMember
 }
 
 
 if ( "Get-UICulture" -eq $_cmdlet )
 {
 $global:_dummy = Get-UICulture
 }
 
 
 if ( "Get-Variable" -eq $_cmdlet )
 {
 $global:_dummy = Get-Variable _
 }
 
 
 if ( "Get-WmiObject" -eq $_cmdlet )
 {
 $global:_dummy = @(iex $str[$i+1])[0]
 }
 
 
 if ( "Group-Object" -eq $_cmdlet )
 {
 $global:_dummy = 1 | group
 }
 
 
 if ( "Measure-Command" -eq $_cmdlet )
 {
 $global:_dummy = Measure-Command {}
 }
 
 
 if ( "Measure-Object" -eq $_cmdlet )
 {
 $global:_dummy = Measure-Object
 }
 
 
 if ( "New-PSDrive" -eq $_cmdlet )
 {
 $global:_dummy =  Get-PSDrive Alias
 }
 
 
 if ( "New-TimeSpan" -eq $_cmdlet )
 {
 $global:_dummy = New-TimeSpan
 }
 
 
 if ( "Resolve-Path" -eq $_cmdlet )
 {
 $global:_dummy = $PWD
 }
 
 
 if ( "Select-String" -eq $_cmdlet )
 {
 $global:_dummy = " " | Select-String " "
 }
 
 
 if ( "Set-Date" -eq $_cmdlet )
 {
 $global:_dummy =  Get-Date
 }
 
 if ( $property -ne $null)
 {
 foreach ( $name in $property.Split(";", "RemoveEmptyEntries" -as [System.StringSplitOptions]) )
 {
 $global:_dummy = @($global:_dummy.$name)[0]
 }
 }
 }
 
 
 function TabExpansion {
 
 param($line, $lastWord)
 
 & {
 function Write-Members ($sep='.')
 {
 Invoke-Expression ('$_val=' + $_expression)
 
 if ( $_expression -match '^\$global:_dummy' )
 {
 $temp = $_expression -replace '^\$global:_dummy(.*)','$1'
 $_expression = '$_' + $temp
 }
 
 $_method = [Management.Automation.PSMemberTypes] `
 'Method,CodeMethod,ScriptMethod,ParameterizedProperty'
 if ($sep -eq '.')
 {
 $params = @{view = 'extended','adapted','base'}
 }
 else
 {
 $params = @{static=$true}
 }
 
 if ( $_val -is [Hashtable] )
 {
 [Object[]]$_keys = $null
 foreach ( $_name in $_val.Keys )
 {
 $_keys += `
 New-Object Microsoft.PowerShell.Commands.MemberDefinition `
 [int],$_name,"Property",0
 }
 }
 
 if ( $_keys -ne $null )
 {
 $_members = [Object[]](Get-Member @params -InputObject $_val $_pat) + ($_keys | ? {$_.name -like $_pat})
 } else {
 $_members = (Get-Member @params -InputObject $_val $_pat)
 }
 
 foreach ($_m in $_members | Sort-Object membertype,name)
 {
 if ($_m.MemberType -band $_method)
 {
 $_base + $_expression + $sep + $_m.name + '('
 }
 else {
 $_base + $_expression + $sep + $_m.name
 }
 }
 }
 $host.UI.RawUI.WindowTitle = "Windows PowerShell V2 (CTP2)" + $lastword
 
 switch ([int]$line[-1])
 {
 4 {
 "[DateTime]::Now"
 [DateTime]::Now
 [DateTime]::Now.ToString("yyyyMMdd")
 [DateTime]::Now.ToString("MMddyyyy")
 [DateTime]::Now.ToString("yyyyMMddHHmmss")
 [DateTime]::Now.ToString("MMddyyyyHHmmss")
 'd f g m o r t u y'.Split(" ") | % { [DateTime]::Now.ToString($_) }
 break;
 }
 
 16 {
 $_base = $lastword.SubString(0, $lastword.Length-1)
 $_base + $global:_cmdstack.Pop()
 break;
 }
 
 18 {
 '$Host.UI.RawUI.'
 '$Host.UI.RawUI'
 break;
 }
 
 22 {
 $_base = $lastword.SubString(0, $lastword.Length-1)
 $global:_clip = New-Object System.Windows.Forms.TextBox
 $global:_clip.Multiline = $true
 $global:_clip.Paste()
 $_base + $global:_clip.Text
 break;
 }
 }
 
 
 switch -regex ($lastWord)
 {
 '(^.*)(\$_\.)(\w*)$' {
 $_base = $matches[1]
 $_expression = '$global:_dummy'
 $_pat = $matches[3] + '*'
 $global:_dummy = $null
 Get-PipeLineObject
 if ( $global:_dummy -eq $null )
 {
 if ( $global:_exp -match '^\$.*\(.*$' )
 {
 $type = ( iex $global:_exp.Split("(")[0] ).OverloadDefinitions[0].Split(" ")[0] -replace '\[[^\[\]]*\]$' -as [type]
 
 if ( $_expression -match '^\$global:_dummy' )
 {
 $temp = $_expression -replace '^\$global:_dummy(.*)','$1'
 $_expression = '$_' + $temp
 }
 
 foreach ( $_m in $type.GetMembers() | sort membertype,name | group name | ? { $_.Name -like $_pat } | % { $_.Group[0] } )
 {
 if ($_m.MemberType -eq "Method")
 {
 $_base + $_expression + '.' + $_m.name + '('
 }
 else {
 $_base + $_expression + '.' + $_m.name
 }
 }
 break;
 }
 elseif ( $global:_exp -match '^\[.*\:\:.*\(.*$' )
 {
 $tname, $mname = $_exp.Split(":(", "RemoveEmptyEntries"-as [System.StringSplitOptions])[0,1]
 $type = @(iex ($tname + '.GetMember("' + $mname + '")'))[0].ReturnType.FullName -replace '\[[^\[\]]*\]$' -as [type]
 
 if ( $_expression -match '^\$global:_dummy' )
 {
 $temp = $_expression -replace '^\$global:_dummy(.*)','$1'
 $_expression = '$_' + $temp
 }
 
 foreach ( $_m in $type.GetMembers() | sort membertype,name | group name | ? { $_.Name -like $_pat } | % { $_.Group[0] } )
 {
 if ($_m.MemberType -eq "Method")
 {
 $_base + $_expression + '.' + $_m.name + '('
 }
 else {
 $_base + $_expression + '.' + $_m.name
 }
 }
 break;
 }
 elseif ( $global:_exp -match '^(\$\w+(\[[0-9,\.]+\])*(\.\w+(\[[0-9,\.]+\])*)*)$' )
 {
 $global:_dummy = @(iex $Matches[1])[0]
 }
 else
 {
 $global:_dummy =  $global:_mix
 }
 }
 
 Write-Members
 break;
 }
 
 '(^.*)(\$(\w|\.)+)\.(\w*)$' {
 $_base = $matches[1]
 $_expression = $matches[2]
 $_pat = $matches[4] + '*'
 if ( $_expression -match '^\$_\.' )
 {
 $_expression = $_expression -replace '^\$_(.*)',('$global:_dummy' + '$1')
 }
 Write-Members
 break;
 }
 
 '(^.*)(\[(\w|\.)+\])\:\:(\w*)$' {
 $_base = $matches[1]
 $_expression = $matches[2]
 $_pat = $matches[4] + '*'
 Write-Members '::'
 break;
 }
 
 '(^.*)(\[(\w|\.)+\]\:\:(\w+\.)+)(\w*)$' {
 Write-Members
 break;
 }
 
 '(^.*\$)(\w+)$' {
 $_prefix = $matches[1]
 $_varName = $matches[2]
 foreach ($_v in Get-ChildItem ('variable:' + $_varName + '*'))
 {
 $_prefix + $_v.name
 }
 break;
 }
 
 '(^.*\$)(.*\:)(\w+)$' {
 $_prefix = $matches[1]
 $_drive = $matches[2]
 $_varName = $matches[3]
 if ($_drive -eq "env:" -or $_drive -eq "function:")
 {
 foreach ($_v in Get-ChildItem ($_drive + $_varName + '*'))
 {
 $_prefix + $_drive + $_v.name
 }
 }
 break;
 }
 
 '(^.*)(\$((\w+\.)|(\w+(\[(\w|,)+\])+\.))+)(\w*)$'
 {
 $_base = $matches[1]
 $_expression = $matches[2].TrimEnd('.')
 $_pat = $Matches[8] + '*'
 if ( $_expression -match '^\$_\.' )
 {
 $_expression = $_expression -replace '^\$_(.*)',('$global:_dummy' + '$1')
 }
 Write-Members
 break;
 }
 
 '(^\[(\w|\.)+\])\.(\w*)$'
 {
 if ( $(iex $Matches[1]) -isnot [System.Type] ) { break; }
 $_expression = $Matches[1]
 $_pat = $Matches[$matches.Count-1] + '*'
 Write-Members
 break;
 }
 
 '^(\[(\w|\.)+\]\.(\w+\.)+)(\w*)$' {
 if ( $(iex $_expression) -eq $null ) { break; }
 Write-Members
 break;
 }
 
 '^(.*)\)((\w|\.)*)\.(\w*)$' {
 $_base = $Matches[1] + ")"
 if ( $matches[3] -eq $null) { $_expression = '[System.Type]' }
 else { $_expression = '[System.Type]' + $Matches[2] }
 $_pat = $matches[4] + '*'
 iex "$_expression | Get-Member $_pat | sort MemberType,Name" |
 % {
 if ( $_.MemberType -like "*Method*" -or $_.MemberType -like "*Parameterized*" ) { $parenthes = "(" }
 if ( $Matches[2] -eq "" ) { $_base + "." + $_.Name + $parenthes }
 else { $_base + $Matches[2] + "." + $_.Name + $parenthes }
 }
 break;
 }
 
 '^\[(\w+(\.\w*)*)$' {
 $_opt = $matches[1] + '*'
 if ( $_opt -eq "*" )
 {
 $_TypeAccelerators -like $_opt -replace '^(.*)$', '[$1]'
 }
 else
 {
 $_TypeAccelerators -like $_opt -replace '^(.*)$', '[$1]'
 "CmdletBinding", "Parameter", "Alias",
 "ValidateArguments", "ValidateCount", "ValidateEnumeratedArguments", "ValidateLength",
 "ValidateNotNull", "ValidateNotNullOrEmpty", "ValidatePattern", "ValidateRange",
 "ValidateScript", "ValidateSet", "AllowEmptyCollection", "AllowEmptyString", "AllowNull" `
 -like $_opt -replace '^(.*)$', '[$1('
 Write-ClassNames $_TypeNames_System ($_opt.Split(".").Count-1) '['
 Write-ClassNames $_TypeNames ($_opt.Split(".").Count-1) '['
 }
 break;
 }
 
 '^\$(env:)?\w+([\\/][^\\/]*)*$' {
 $path = iex ('"' + $Matches[0] + '"')
 if ( $Matches[2].Length -gt 1 )
 {
 $parent = Split-Path $path -Parent
 $leaf = (Split-Path $path -Leaf) + '*'
 }
 else
 {
 $parent = $path
 $leaf = '*'
 }
 if ( Test-Path $parent )
 {
 $i = $Matches[0].LastIndexOfAny("/\")
 $_base = $Matches[0].Substring(0,$i+1)
 [IO.Directory]::GetFileSystemEntries( $parent, $leaf ) | % { $_base + ($_.Split("\/")[-1] -replace '([\$\s&])','`$1' -replace '([[\]])', '````$1') }
 }
 }
 
 '^(\^?([^~]+))(~(.*))*@$' {
 if ( $Matches[1] -notlike "^*" )
 {
 $include = $Matches[2] -replace '``','`'
 if ( $Matches[3] )
 {
 $exclude = $Matches[3].Split("~", "RemoveEmptyEntries" -as [System.StringSplitOptions]) -replace '``','`'
 }
 }
 else
 {
 $include = "*"
 $exclude = $Matches[2] -replace '``','`'
 }
 $fse = [IO.Directory]::GetFileSystemEntries($PWD)
 $fse = $fse -replace '.*[\\/]([^/\\]*)$','$1'
 % -in ($fse -like $include) { $fse = $_; $exclude | % { $fse = $fse -notlike $_ } }
 $fse = $fse -replace '^.*\s.*$', ("'`$0'")
 $fse = $fse -replace '([\[\]])', '``$1' -replace '^.*([\[\]]).*$', ("'`$0'")
 $fse = $fse -replace "''", "'"
 $OFS = ", "; "$fse"
 $OFS = ", "; "* -Filter $include " + $(if($exclude){"-Exclude $exclude"})
 $Matches[0].Substring(0, $Matches[0].Length-1)
 break;
 }
 
 '(.*);(.?)$' {
 $_base = $Matches[1]
 if ( $Matches[2] -eq ":" -or $Matches[2] -eq "," )
 {
 if ( $_cmdstack.Count -gt 0 )
 {
 $_base + $global:_cmdstack.Pop()
 }
 else
 {
 ""; break;
 }
 }
 elseif ( $Matches[2] -eq "" )
 {
 $global:_cmdstack.Push($line.SubString(0,$line.Length-1))
 [System.Windows.Forms.SendKeys]::SendWait("{ESC}")
 ""; break;
 }
 }
 
 '^-([\w0-9]*)' {
 $_pat = $matches[1] + '*'
 
 $_command = [regex]::Split($line, '[|;=]')[-1]
 
 if ($_command -match '\{([^\{\}]*)$')
 {
 $_command = $matches[1]
 }
 
 if ($_command -match '\(([^()]*)$')
 {
 $_command = $matches[1]
 }
 
 $_command = $_command.Trim().Split()[0]
 
 $_command = @(Get-Command -type 'All' $_command)[0]
 
 while ($_command.CommandType -eq 'alias')
 {
 $_command = @(Get-Command -type 'All' $_command.Definition)[0]
 }
 
 if ( $_command.name -eq "powershell.exe" )
 {
 if ( $global:_PSexeOption )
 {
 $global:_PSexeOption -like "-$_pat" -replace '^(-[^,]+).*$', '$1' | sort
 }
 else
 {
 ($global:_PSexeOption = powershell.exe -?) -like "-$_pat" -replace '^(-[^,]+).*$', '$1' | sort
 }
 break;
 }
 
 
 if ( $_command -ne $null )
 {
 foreach ($_n in $_command.Parameters.Keys | sort)
 {
 if ($_n -like $_pat) { '-' + $_n }
 }
 }
 elseif ( $line -match 'switch\s+(-\w+\s+)*-(\w*)$')
 {
 $_pat = $Matches[2] + '*'
 "regex", "wildcard", "exact", "casesensitive", "file" -like $_pat -replace '^(.*)$', '-$1'
 break;
 }
 else
 {
 "-and", "-as", "-band", "-bnot", "-bor", "-bxor", "-ccontains", "-ceq", "-cge", "-cgt", "-cle", "-clike", "-clt",
 "-cmatch", "-cne", "-cnotcontains", "-cnotlike", "-cnotmatch", "-contains", "-creplace", "-csplit", "-eq", "-f", "-ge",
 "-gt", "-icontains", "-ieq", "-ige", "-igt", "-ile", "-ilike", "-ilt", "-imatch", "-ine", "-inotcontains", "-inotlike",
 "-inotmatch", "-ireplace", "-is", "-isnot", "-isplit", "-join", "-le", "-like", "-lt", "-match", "-ne", "-not", "-notcontains", 
 "-notlike", "-notmatch", "-or", "-replace", "-split", "-xor" -like "-$_pat"
 }
 break;
 }
 
 '^#(\w*)' {
 $_pattern = $matches[1]
 if ($_pattern -match '^[0-9]+$')
 {
 Get-History -ea SilentlyContinue -Id $_pattern | Foreach { $_.CommandLine } 
 }
 else
 {
 $_pattern = '*' + $_pattern + '*'
 Get-History | Sort-Object -Descending Id| Foreach { $_.CommandLine } | where { $_ -like $_pattern }
 }
 break;
 }
 
 default {
 
 $_tokens = [System.Management.Automation.PSParser]::Tokenize($line, [ref] $null)
 
 if ( $_tokens )
 {
 $_lastToken = $_tokens[$_tokens.count - 1]
 if ($_lastToken.Type -eq 'Member')
 {
 $_pat = $_lastToken.Content + '*'
 $i=$_tokens.count; do { $i-- } until ( $_tokens[$i].Type -eq "Attribute")
 if ( $lastWord -match "^(.*)([\(,])\w*$" )
 {
 $_base = $matches[1] + $matches[2]
 }
 switch ( $_tokens[$i].Content )
 {
 'Parameter' {
 [System.Management.Automation.ParameterAttribute].GetProperties() | ? { $_.Name -like $_pat -and $_.Name -ne "TypeId" } | % { $_base + $_.Name + "=" }
 }
 'CmdletBinding' {
 [System.Management.Automation.CmdletBindingAttribute].GetProperties() | ? { $_.Name -like $_pat -and $_.Name -ne "TypeId" } | % { $_base + $_.Name + "=" }
 }
 }
 break;
 }
 
 if ( $_tokens[1].Type -eq "Attribute")
 {
 if ( $line.Split("(").Count -gt $line.Split(")").Count )
 {
 if ( $lastWord -match "^(.*)([\(,])\w*$" )
 {
 $_base = $matches[1] + $matches[2]
 }
 switch ( $_tokens[1].Content )
 {
 'Parameter' {
 [System.Management.Automation.ParameterAttribute].GetProperties() | % { $_base + $_.Name + "=" }
 }
 'CmdletBinding' {
 [System.Management.Automation.CmdletBindingAttribute].GetProperties()  | % { $_base + $_.Name + "=" }
 }
 }
 }
 break;
 }
 }
 
 
 $lastex =  [regex]::Split($line, '[|;]')[-1]
 if ( $lastex -match '^\s*(\$\w+(\[[0-9,]+\])*(\.\w+(\[[0-9,]+\])*)*)\s*=\s+(("\w+"\s*,\s+)*)"\w+"\s*-as\s+$' )
 {
 if ( $Matches[6] -ne $nul )
 {
 $brackets = "[]"
 }
 '['+ $global:_enum + $brackets + ']'
 break;
 }
 
 
 if ( $lastex -match '^\s*(\$\w+(\[[0-9,]+\])*(\.\w+(\[[0-9,]+\])*)*)\s*=\s+(("\w+"\s*,\s+)*)\s*(\w*)$' )
 {
 $_pat = $Matches[7] + '*'
 
 $_type = @(iex $Matches[1])[0].GetType()
 if ( $_type.IsEnum )
 {
 $global:_enum = $_type.FullName
 [Enum]::GetValues($_type) -like $_pat -replace '^(.*)$','"$1"'
 break;
 }
 }
 
 $lastex =  [regex]::Split($line, '[|;=]')[-1]
 if ($lastex  -match '[[$].*\w+\(.*-as\s*$')
 {
 '['+ $global:_enum + ']'
 }
 elseif ( $lastex -match '([[$].*(\w+))\((.*)$' )
 {
 $_method = $Matches[1]
 
 if ( $Matches[3] -match "(.*)((`"|')(\w+,)+(\w*))$" )
 {
 $continuous = $true
 $_opt =  $Matches[5] + '*'
 $_base =  $Matches[2].TrimStart('"') -replace '(.*,)\w+$','$1'
 $position = $Matches[1].Split(",").Length
 }
 else
 {
 $continuous = $false
 $_opt = ($Matches[3].Split(',')[-1] -replace '^\s*','') + "*"
 $position = $Matches[3].Split(",").Length
 }
 
 if ( ($_mdefs = iex ($_method + ".OverloadDefinitions")) -eq $null )
 {
 $tname, $mname = $_method.Split(":", "RemoveEmptyEntries" -as [System.StringSplitOptions])
 $_mdefs = iex ($tname + '.GetMember("' + $mname + '") | % { $_.ToString() }')
 }
 
 foreach ( $def in $_mdefs )
 {
 [void] ($def -match '\((.*)\)')
 foreach ( $param in [regex]::Split($Matches[1], ', ')[$position-1] )
 {
 if ($param -eq $null -or $param -eq "")
 {
 continue;
 }
 $type = $param.split()[0]
 
 if ( $type -like '*`[*' -or $type -eq "Params" -or $type -eq "" )
 {
 continue;
 }
 $fullname  = @($_typenames -like "*$type*")
 foreach ( $name in $fullname )
 {
 if ( $continuous -eq $true -and ( $name  -as [System.Type] ).IsEnum )
 {
 $output = [Enum]::GetValues($name) -like $_opt -replace '^(.*)$',($_base + '$1')
 $output | sort
 }
 elseif ( ( $name  -as [System.Type] ).IsEnum ) 
 {
 $global:_enum = $name
 $output = [Enum]::GetValues($name) -like $_opt -replace '^(.*)$','"$1"'
 $output | sort
 }
 }
 }
 }
 if ( $output -ne $null )
 {
 break;
 }
 }
 
 if ( $line -match '(function|filter)\s+(\w*)$')
 {
 $_pat = 'function:\' + $Matches[2] + '*'
 Get-ChildItem $_pat| % { $_.Name }
 break;
 }
 
 
 if ( $line[-1] -eq " " )
 {
 $_cmdlet = $line.TrimEnd(" ").Split(" |(;={")[-1]
 
 $_cmdlet = @(Get-Command -type 'cmdlet,alias,function' $_cmdlet)[0]
 
 while ($_cmdlet.CommandType -eq 'alias')
 {
 $_cmdlet = @(Get-Command -type 'cmdlet,alias,function' $_cmdlet.Definition)[0]
 }
 
 if ( "Set-ExecutionPolicy" -eq $_cmdlet.Name )
 {
 "Unrestricted", "RemoteSigned", "AllSigned", "Restricted", "Default" | sort
 break;
 }
 
 if ( "Trace-Command","Get-TraceSource","Set-TraceSource" -contains $_cmdlet.Name )
 {
 Get-TraceSource | % { $_.Name } | sort -Unique
 break;
 }
 
 if ( "New-Object" -eq $_cmdlet.Name )
 {
 $_TypeAccelerators
 break;
 }
 
 if ( $_cmdlet.Noun -like "*WMI*" )
 {
 $_WMIClasses
 break;
 }
 
 if ( "Get-Process" -eq $_cmdlet.Name )
 {
 Get-Process | % { $_.Name } | sort
 break;
 }
 
 if ( "Add-PSSnapin", "Get-PSSnapin", "Remove-PSSnapin" -contains $_cmdlet.Name )
 {
 if ( $global:_snapin -ne $null )
 {
 $global:_snapin
 break;
 }
 else
 {
 $global:_snapin = $(Get-PSSnapIn -Registered;Get-PSSnapIn)| sort Name -Unique;
 $global:_snapin
 break;
 }
 }
 
 if ( "Get-PSDrive", "New-PSDrive", "Remove-PSDrive" `
 -contains $_cmdlet.Name -and "Name" )
 {
 Get-PSDrive | sort
 break;
 }
 
 if ( "Get-Eventlog" -eq $_cmdlet.Name )
 {
 Get-EventLog -List | % { $_base + ($_.Log -replace '\s','` ') }
 break;
 }
 
 if ( "Get-Help" -eq $_cmdlet.Name -or "help" -eq $_cmdlet.Name )
 {
 Get-Help -Category all | % { $_.Name } | sort -Unique
 break;
 }
 
 if ( "Get-Service", "Restart-Service", "Resume-Service",
 "Start-Service", "Stop-Service", "Suspend-Service" `
 -contains $_cmdlet.Name )
 {
 Get-Service | sort Name  | % { $_base + ($_.Name -replace '\s','` ') }
 break;
 }
 
 if ( "Get-Command" -eq $_cmdlet.Name )
 {
 Get-Command -CommandType All | % { $_base + ($_.Name -replace '\s','` ') }
 break;
 }
 
 if ( "Format-List", "Format-Custom", "Format-Table", "Format-Wide", "Compare-Object",
 "ConvertTo-Html", "Measure-Object", "Select-Object", "Group-Object", "Sort-Object" `
 -contains $_cmdlet.Name )
 {
 Get-PipeLineObject
 $_dummy | gm -MemberType Properties,ParameterizedProperty | sort membertype | % { $_base + ($_.Name -replace '\s','` ') }
 break;
 }
 
 if ( "Clear-Variable", "Get-Variable", "New-Variable", "Remove-Variable", "Set-Variable" -contains $_cmdlet.Name )
 {
 Get-Variable -Scope Global | % { $_.Name } | sort | % { $_base + ($_ -replace '\s','` ') }
 break;
 }
 
 if ( "Get-Alias", "New-Alias", "Set-Alias" -contains $_cmdlet.Name )
 {
 Get-Alias | % { $_.Name } | sort | % { $_base + ($_ -replace '\s','` ') }
 break;
 }
 }
 
 
 if ( $line[-1] -eq " " )
 {
 $_cmdlet = [regex]::Split($line, '[|;=]')[-1]
 
 if ($_cmdlet -match '\{([^\{\}]*)$')
 {
 $_cmdlet = $matches[1]
 }
 
 if ($_cmdlet -match '\(([^()]*)$')
 {
 $_cmdlet = $matches[1]
 }
 
 $_cmdlet = $_cmdlet.Trim().Split()[0]
 
 $_cmdlet = @(Get-Command -type 'Application' $_cmdlet)[0]
 
 if ( $_cmdlet.Name -eq "powershell.exe" )
 {
 "-PSConsoleFile", "-Version", "-NoLogo", "-NoExit", "-Sta", "-NoProfile", "-NonInteractive",
 "-InputFormat", "-OutputFormat", "-EncodedCommand", "-File", "-Command" | sort
 break;
 }
 if ( $_cmdlet.Name -eq "fsutil.exe" )
 {
 "behavior query", "behavior set", "dirty query", "dirty set", 
 "file findbysid", "file queryallocranges", "file setshortname", "file setvaliddata", "file setzerodata", "file createnew", 
 "fsinfo drives", "fsinfo drivetype", "fsinfo volumeinfo", "fsinfo ntfsinfo", "fsinfo statistics", 
 "hardlink create", "objectid query", "objectid set", "objectid delete", "objectid create",
 "quota disable", "quota track", "quota enforce", "quota violations", "quota modify", "quota query",
 "reparsepoint query", "reparsepoint delete", "sparse setflag", "sparse queryflag", "sparse queryrange", "sparse setrange",
 "usn createjournal", "usn deletejournal", "usn enumdata", "usn queryjournal", "usn readdata", "volume dismount", "volume diskfree" | sort
 break;
 }
 if ( $_cmdlet.Name -eq "net.exe" )
 {
 "ACCOUNTS ", " COMPUTER ", " CONFIG ", " CONTINUE ", " FILE ", " GROUP ", " HELP ", 
 "HELPMSG ", " LOCALGROUP ", " NAME ", " PAUSE ", " PRINT ", " SEND ", " SESSION ", 
 "SHARE ", " START ", " STATISTICS ", " STOP ", " TIME ", " USE ", " USER ", " VIEW" | sort
 break;
 }
 if ( $_cmdlet.Name -eq "ipconfig.exe" )
 {
 "/?", "/all", "/renew", "/release", "/flushdns", "/displaydns",
 "/registerdns", "/showclassid", "/setclassid"
 break;
 }
 }
 
 if ( $line -match '\w+\s+(\w+(\.|[^\s\.])*)$' )
 {
 #$_opt = $Matches[1] + '*'
 $_cmdlet = $line.TrimEnd(" ").Split(" |(;={")[-2]
 
 $_opt = $Matches[1].Split(" ,")[-1] + '*'
 $_base = $Matches[1].Substring(0,$Matches[1].Length-$Matches[1].Split(" ,")[-1].length)
 
 
 $_cmdlet = @(Get-Command -type 'cmdlet,alias,function' $_cmdlet)[0]
 
 while ($_cmdlet.CommandType -eq 'alias')
 {
 $_cmdlet = @(Get-Command -type 'cmdlet,alias,function' $_cmdlet.Definition)[0]
 }
 
 if ( "Set-ExecutionPolicy" -eq $_cmdlet.Name )
 {
 "Unrestricted", "RemoteSigned", "AllSigned", "Restricted", "Default" -like $_opt | sort
 break;
 }
 
 if ( "Trace-Command","Get-TraceSource","Set-TraceSource" -contains $_cmdlet.Name )
 {
 Get-TraceSource -Name $_opt | % { $_.Name } | sort -Unique | % { $_base + ($_ -replace '\s','` ') }
 break;
 }
 
 if ( "New-Object" -eq $_cmdlet.Name )
 {
 $_TypeAccelerators -like $_opt
 Write-ClassNames $_TypeNames_System ($_opt.Split(".").Count-1)
 Write-ClassNames $_TypeNames ($_opt.Split(".").Count-1)
 break;
 }
 
 if ( $_cmdlet.Name -like "*WMI*" )
 {
 Write-ClassNames $_WMIClasses ($_opt.Split("_").Count-1) -sep '_'
 break;
 }
 
 if ( "Get-Process" -eq $_cmdlet.Name )
 {
 Get-Process $_opt | % { $_.Name } | sort | % { $_base + ($_ -replace '\s','` ') }
 break;
 }
 
 if ( "Add-PSSnapin", "Get-PSSnapin", "Remove-PSSnapin" -contains $_cmdlet.Name )
 {
 if ( $global:_snapin -ne $null )
 {
 $global:_snapin -like $_opt | % { $_base + ($_ -replace '\s','` ') }
 break;
 }
 else
 {
 $global:_snapin = $(Get-PSSnapIn -Registered;Get-PSSnapIn)| sort Name -Unique;
 $global:_snapin -like $_opt | % { $_base + ($_ -replace '\s','` ') }
 break;
 }
 }
 
 if ( "Get-PSDrive", "New-PSDrive", "Remove-PSDrive" `
 -contains $_cmdlet.Name -and "Name" )
 {
 Get-PSDrive -Name $_opt | sort | % { $_base + ($_ -replace '\s','` ') }
 break;
 }
 
 if ( "Get-PSProvider" -eq $_cmdlet.Name )
 {
 Get-PSProvider -PSProvider $_opt | % { $_.Name } | sort | % { $_base + ($_ -replace '\s','` ') }
 break;
 }
 
 
 if ( "Get-Eventlog" -eq $_cmdlet.Name )
 {
 Get-EventLog -List | ? { $_.Log -like $_opt } | % { $_base + ($_.Log -replace '\s','` ') }
 break;
 }
 
 if ( "Get-Help" -eq $_cmdlet.Name -or "help" -eq $_cmdlet.Name )
 {
 Get-Help -Category all -Name $_opt | % { $_.Name } | sort -Unique
 break;
 }
 
 if ( "Get-Service", "Restart-Service", "Resume-Service",
 "Start-Service", "Stop-Service", "Suspend-Service" `
 -contains $_cmdlet.Name )
 {
 Get-Service -Name $_opt | sort Name  | % { $_base + ($_.Name -replace '\s','` ') }
 break;
 }
 
 if ( "Get-Command" -eq $_cmdlet.Name )
 {
 Get-Command -CommandType All -Name $_opt | % { $_base + ($_.Name -replace '\s','` ') }
 break;
 }
 
 if ( "Format-List", "Format-Custom", "Format-Table", "Format-Wide", "Compare-Object",
 "ConvertTo-Html", "Measure-Object", "Select-Object", "Group-Object", "Sort-Object" `
 -contains $_cmdlet.Name )
 {
 
 Get-PipeLineObject
 $_dummy | Get-Member -Name $_opt -MemberType Properties,ParameterizedProperty | sort membertype | % { $_base + ($_.Name -replace '\s','` ') }
 break;
 }
 
 if ( "Clear-Variable", "Get-Variable", "New-Variable", "Remove-Variable", "Set-Variable" -contains $_cmdlet.Name )
 {
 Get-Variable -Scope Global -Name $_opt | % { $_.Name } | sort | % { $_base + ($_ -replace '\s','` ') }
 break;
 }
 
 if ( "Get-Alias", "New-Alias", "Set-Alias" -contains $_cmdlet.Name )
 {
 Get-Alias -Name $_opt | % { $_.Name } | sort | % { $_base + ($_ -replace '\s','` ') }
 break;
 }
 }
 
 if ( $line -match '(-(\w+))\s+([^-]*$)' )
 {
 
 $_param = $matches[2] + '*'
 $_opt = $Matches[3].Split(" ,")[-1] + '*'
 $_base = $Matches[3].Substring(0,$Matches[3].Length-$Matches[3].Split(" ,")[-1].length)
 #$_opt = ($Matches[3] -replace '(^.*\s*,?\s*)\w*$','$1') + '*'
 
 $_cmdlet = [regex]::Split($line, '[|;=]')[-1]
 
 if ($_cmdlet -match '\{([^\{\}]*)$')
 {
 $_cmdlet = $matches[1]
 }
 
 if ($_cmdlet -match '\(([^()]*)$')
 {
 $_cmdlet = $matches[1]
 }
 
 $_cmdlet = $_cmdlet.Trim().Split()[0]
 
 $_cmdlet = @(Get-Command -type 'Cmdlet,Alias,Function,Filter,ExternalScript' $_cmdlet)[0]
 
 while ($_cmdlet.CommandType -eq 'alias')
 {
 $_cmdlet = @(Get-Command -type 'Cmdlet,Alias,Function,Filter,ExternalScript' $_cmdlet.Definition)[0]
 }
 
 if ( $_param.TrimEnd("*") -eq "ea" -or $_param.TrimEnd("*") -eq "wa" )
 {
 "SilentlyContinue", "Stop", "Continue", "Inquire" |
 ? { $_ -like $_opt } | sort -Unique
 break;
 }
 
 if ( "Format-List", "Format-Custom", "Format-Table", "Format-Wide" -contains $_cmdlet.Name `
 -and "groupBy" -like $_param )
 {
 Get-PipeLineObject
 $_dummy | Get-Member -Name $_opt -MemberType Properties,ParameterizedProperty | sort membertype | % { $_.Name }
 break;
 }
 
 if ( $_param.TrimEnd("*") -eq "ev" -or $_param.TrimEnd("*") -eq "ov" -or
 "ErrorVariable" -like $_param -or "OutVariable" -like $_param)
 {
 Get-Variable -Scope Global -Name $_opt | % { $_.Name } | sort
 break;
 }
 
 if ( "Tee-Object" -eq $_cmdlet.Name -and "Variable" -like $_param )
 {
 Get-Variable -Scope Global -Name $_opt | % { $_.Name } | sort
 break;
 }
 
 if ( "Clear-Variable", "Get-Variable", "New-Variable", "Remove-Variable", "Set-Variable" -contains $_cmdlet.Name `
 -and "Name" -like $_param)
 {
 Get-Variable -Scope Global -Name $_opt | % { $_.Name } | sort | % { $_base + ($_ -replace '\s','` ') }
 break;
 }
 
 if ( "Export-Alias", "Get-Alias", "New-Alias", "Set-Alias" -contains $_cmdlet.Name `
 -and "Name" -like $_param)
 {
 Get-Alias -Name $_opt | % { $_.Name } | sort | % { $_base + ($_ -replace '\s','` ') }
 break;
 }
 
 if ( "Out-File","Export-CSV","Select-String","Export-Clixml" -contains $_cmdlet.Name `
 -and "Encoding" -like $_param)
 {
 "Unicode",  "UTF7", "UTF8", "ASCII", "UTF32", "BigEndianUnicode", "Default", "OEM" |
 ? { $_ -like $_opt } | sort -Unique
 break;
 }
 
 if ( "Trace-Command","Get-TraceSource","Set-TraceSource" -contains $_cmdlet.Name `
 -and "Name" -like $_param)
 {
 Get-TraceSource -Name $_opt | % { $_.Name } | sort -Unique | % { $_base + ($_ -replace '\s','` ') }
 break;
 }
 
 if ( "New-Object" -like $_cmdlet.Name )
 {
 if ( "ComObject" -like $_param )
 {
 $_ProgID -like $_opt | % { $_ -replace '\s','` ' }
 break;
 }
 
 if ( "TypeName" -like $_param )
 {
 if ( $_opt -eq "*" )
 {
 $_TypeAccelerators -like $_opt
 }
 else
 {
 $_TypeAccelerators -like $_opt
 Write-ClassNames $_TypeNames_System ($_opt.Split(".").Count-1)
 Write-ClassNames $_TypeNames ($_opt.Split(".").Count-1)
 }
 break;
 }
 }
 
 if ( "New-Item" -eq $_cmdlet.Name )
 {
 if ( "ItemType" -like $_param )
 {
 "directory", "file" -like $_opt
 break;
 }
 }
 
 if ( "Get-Location", "Get-PSDrive", "Get-PSProvider", "New-PSDrive", "Remove-PSDrive" `
 -contains $_cmdlet.Name `
 -and "PSProvider" -like $_param )
 {
 Get-PSProvider -PSProvider $_opt | % { $_.Name } | sort  | % { $_base + ($_ -replace '\s','` ') }
 break;
 }
 
 if ( "Get-Location" -eq $_cmdlet.Name -and "PSDrive" -like $_param )
 {
 Get-PSDrive -Name $_opt | sort | % { $_base + ($_ -replace '\s','` ') }
 break;
 }
 
 if ( "Get-PSDrive", "New-PSDrive", "Remove-PSDrive" `
 -contains $_cmdlet.Name -and "Name" -like $_param )
 {
 Get-PSDrive -Name $_opt | sort | % { $_base + ($_ -replace '\s','` ') }
 break;
 }
 
 if ( "Get-Command" -eq $_cmdlet.Name -and  "PSSnapin" -like $_param)
 {
 if ( $global:_snapin -ne $null )
 {
 $global:_snapin -like $_opt  | % { $_base + $_ }
 break;
 }
 else
 {
 $global:_snapin = $(Get-PSSnapIn -Registered;Get-PSSnapIn)| sort Name -Unique;
 $global:_snapin -like $_opt  | % { $_base + ($_ -replace '\s','` ') }
 break;
 }
 }
 
 if ( "Add-PSSnapin", "Get-PSSnapin", "Remove-PSSnapin" `
 -contains $_cmdlet.Name -and "Name" -like $_param )
 {
 if ( $global:_snapin -ne $null )
 {
 $global:_snapin -like $_opt | % { $_base + ($_ -replace '\s','` ') }
 break;
 }
 else
 {
 $global:_snapin = $(Get-PSSnapIn -Registered;Get-PSSnapIn)| sort Name -Unique;
 $global:_snapin -like $_opt | % { $_base + $_ }
 break;
 }
 }
 
 if ( "Clear-Variable", "Export-Alias", "Get-Alias", "Get-PSDrive", "Get-Variable", "Import-Alias",
 "New-Alias", "New-PSDrive", "New-Variable", "Remove-Variable", "Set-Alias", "Set-Variable" `
 -contains $_cmdlet.Name -and "Scope" -like $_param )
 {
 "Global", "Local", "Script" -like $_opt
 break;
 }
 
 if ( "Get-Process", "Stop-Process", "Wait-Process" -contains $_cmdlet.Name -and "Name" -like $_param )
 {
 Get-Process $_opt | % { $_.Name } | sort | % { $_base + ($_ -replace '\s','` ') }
 break;
 }
 
 if ( "Get-Eventlog" -eq $_cmdlet.Name -and "LogName" -like $_param )
 {
 Get-EventLog -List | ? { $_.Log -like $_opt } | % { $_base + ($_.Log -replace '\s','` ') }
 break;
 }
 
 if ( "Get-Help" -eq $_cmdlet.Name -or "help" -eq $_cmdlet.Name )
 {
 if ( "Name" -like $_param )
 {
 Get-Help -Category all -Name $_opt | % { $_.Name } | sort -Unique
 break;
 }
 if ( "Category" -like $_param )
 {
 "Alias", "Cmdlet", "Provider", "General", "FAQ",
 "Glossary", "HelpFile", "All" -like $_opt | sort | % { $_base + $_ }
 break;
 }
 }
 
 if ( "Get-Service", "Restart-Service", "Resume-Service",
 "Start-Service", "Stop-Service", "Suspend-Service" `
 -contains $_cmdlet.Name )
 {
 if ( "Name" -like $_param )
 {
 Get-Service -Name $_opt | sort Name  | % { $_base + ($_.Name -replace '\s','` ') }
 break;
 }
 if ( "DisplayName" -like $_param )
 {
 Get-Service -Name $_opt | sort DisplayName | % { $_base + ($_.DisplayName -replace '\s','` ') }
 break;
 }
 }
 
 if ( "New-Service" -eq $_cmdlet.Name -and "dependsOn" -like $_param )
 {
 Get-Service -Name $_opt | sort Name | % { $_base + ($_.Name -replace '\s','` ') }
 break;
 }
 
 if ( "Get-EventLog" -eq $_cmdlet.Name -and "EntryType" -like $_param )
 {
 "Error", "Information", "FailureAudit", "SuccessAudit", "Warning" -like $_opt | sort | % { $_base + $_ }
 break;
 }
 
 if ( "Get-Command" -eq $_cmdlet.Name -and "Name" -like $_param )
 {
 Get-Command -CommandType All -Name $_opt | % { $_base + ($_.Name -replace '\s','` ') }
 break;
 }
 
 if ( $_cmdlet.Noun -like "*WMI*" )
 {
 if ( "Class" -like $_param )
 {
 Write-ClassNames $_WMIClasses ($_opt.Split("_").Count-1) -sep '_'
 break;
 }
 }
 
 if ( "Format-List", "Format-Custom", "Format-Table", "Format-Wide", "Compare-Object",
 "ConvertTo-Html", "Measure-Object", "Select-Object", "Group-Object", "Sort-Object" `
 -contains $_cmdlet.Name -and "Property" -like $_param )
 {
 Get-PipeLineObject
 $_dummy | Get-Member -Name $_opt -MemberType Properties,ParameterizedProperty | sort membertype | % { $_base + ($_.Name -replace '\s','` ') }
 break;
 }
 
 if ( "Select-Object" -eq $_cmdlet.Name )
 {
 if ( "ExcludeProperty" -like $_param )
 {
 Get-PipeLineObject
 $_dummy | Get-Member -Name $_opt -MemberType Properties,ParameterizedProperty | sort membertype | % { $_base + ($_.Name -replace '\s','` ') }
 break;
 }
 
 if ( "ExpandProperty" -like $_param )
 {
 Get-PipeLineObject
 $_dummy | Get-Member -Name $_opt -MemberType Properties,ParameterizedProperty | sort membertype | % { $_.Name }
 break;
 }
 }
 
 select -InputObject $_cmdlet -ExpandProperty ParameterSets | select -ExpandProperty Parameters |
 ? { $_.Name -like $_param } | ? { $_.ParameterType.IsEnum } |
 % { [Enum]::GetNames($_.ParameterType) } | ? { $_ -like $_opt } | sort -Unique | % { $_base + $_ }
 
 }
 
 if ($_tokens)
 {
 $_lastToken = $_tokens[$_tokens.count - 1]
 if ($_lastToken.Type -eq 'Command')
 {
 $_cmd = $_lastToken.Content
 
 if ($_cmd.IndexOfAny('/\') -eq -1)
 {
 if ($lastword.substring($lastword.length-$_cmd.length) -eq $_cmd)
 {
 $_pat = $_cmd + '*'
 $_base = $lastword.substring(0, $lastword.length-$_cmd.length)
 "begin {", "break", "catch {", "continue", "data {", "do {", "dynamicparam (", "else {", "elseif (",
 "end {", "exit", "filter ", "finally {", "for (", "foreach ", "from", "function ", "if (", "in ",
 "param (", "process {", "return", "switch ", "throw ", "trap ", "try {", "until (", "while (" `
 -like $_pat | %  {'{0}{1}' -f $_base,$_ }
 $ExecutionContext.InvokeCommand.GetCommandName($_pat,$true, $false) |
 Sort-Object -Unique | ForEach-Object {'{0}{1}' -f $_base,$_ }
 }
 }
 }
 }
 }
 }
 }
 }
`

