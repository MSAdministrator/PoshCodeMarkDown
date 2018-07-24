---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 737
Published Date: 
Archived Date: 2008-12-20t03
---

# get-pipelineobject - 

## Description

for tabexpansion.ps1

## Comments



## Usage



## TODO



## function

`get-pipelineobject`

## Code

`#
 #
 function Get-PipeLineObject {
 
     $i = -2
     do {
         $str = $line.Split("|")[$i]
         $_cmdlet = [regex]::Split($str, '[|;=]')[-1]
 
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
         Add-Member -Name CommandType -MemberType 'NoteProperty' -Value $f.CommandType -PassThru |
         Add-Member -Name Definition -MemberType 'NoteProperty' -Value $a.Definition -PassThru |
         Add-Member -Name Description -MemberType 'NoteProperty' -Value $a.Description -PassThru |
         Add-Member -Name Key -MemberType 'NoteProperty' -Value $e.Key -PassThru |
         Add-Member -Name Location -MemberType 'NoteProperty' -Value $c.Location -PassThru |
         Add-Member -Name LocationName -MemberType 'NoteProperty' -Value $c.LocationName -PassThru |
         Add-Member -Name Options -MemberType 'NoteProperty' -Value $a.Options -PassThru |
         Add-Member -Name ReferencedCommand -MemberType 'NoteProperty' -Value $a.ReferencedCommand -PassThru |
         Add-Member -Name ResolvedCommand -MemberType 'NoteProperty' -Value $a.ResolvedCommand -PassThru |
         Add-Member -Name ScriptBlock -MemberType 'NoteProperty' -Value $f.ScriptBlock -PassThru |
         Add-Member -Name StoreNames -MemberType 'NoteProperty' -Value $c.StoreNames -PassThru |
         Add-Member -Name SubKeyCount -MemberType 'NoteProperty' -Value $h.SubKeyCount -PassThru |
         Add-Member -Name Value -MemberType 'NoteProperty' -Value $e.Value -PassThru |
         Add-Member -Name ValueCount -MemberType 'NoteProperty' -Value $h.ValueCount -PassThru |
         Add-Member -Name Visibility -MemberType 'NoteProperty' -Value $a.Visibility -PassThru |
         Add-Member -Name Property -MemberType 'NoteProperty' -Value $h.Property -PassThru |
         Add-Member -Name ResolvedCommandName -MemberType 'NoteProperty' -Value $a.ResolvedCommandName -PassThru |
         Add-Member -Name Close -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name CreateSubKey -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name DeleteSubKey -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name DeleteSubKeyTree -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name DeleteValue -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name Flush -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetSubKeyNames -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetValue -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetValueKind -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetValueNames -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name IsValidValue -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name OpenSubKey -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name SetValue -MemberType 'ScriptMethod' -Value {} -PassThru
     }
 
     if ( $global:_mix -eq $null )
     {
         $f = gi $PSHOME\powershell.exe
         $t = [type]
         $s = ""
         $global:_mix = `
         Add-Member -InputObject (New-Object PSObject) -Name Mode -MemberType 'NoteProperty' -Value $f.Mode -PassThru |
         Add-Member -Name Assembly -MemberType 'NoteProperty' -Value $t.Assembly -PassThru |
         Add-Member -Name AssemblyQualifiedName -MemberType 'NoteProperty' -Value $t.AssemblyQualifiedName -PassThru |
         Add-Member -Name Attributes -MemberType 'NoteProperty' -Value $f.Attributes -PassThru |
         Add-Member -Name BaseType -MemberType 'NoteProperty' -Value $t.BaseType -PassThru |
         Add-Member -Name ContainsGenericParameters -MemberType 'NoteProperty' -Value $t.ContainsGenericParameters -PassThru |
         Add-Member -Name CreationTime -MemberType 'NoteProperty' -Value $f.CreationTime -PassThru |
         Add-Member -Name CreationTimeUtc -MemberType 'NoteProperty' -Value $f.CreationTimeUtc -PassThru |
         Add-Member -Name DeclaringMethod -MemberType 'NoteProperty' -Value $t.DeclaringMethod -PassThru |
         Add-Member -Name DeclaringType -MemberType 'NoteProperty' -Value $t.DeclaringType -PassThru |
         Add-Member -Name Exists -MemberType 'NoteProperty' -Value $f.Exists -PassThru |
         Add-Member -Name Extension -MemberType 'NoteProperty' -Value $f.Extension -PassThru |
         Add-Member -Name FullName -MemberType 'NoteProperty' -Value $f.FullName -PassThru |
         Add-Member -Name GenericParameterAttributes -MemberType 'NoteProperty' -Value $t.GenericParameterAttributes -PassThru |
         Add-Member -Name GenericParameterPosition -MemberType 'NoteProperty' -Value $t.GenericParameterPosition -PassThru |
         Add-Member -Name GUID -MemberType 'NoteProperty' -Value $t.GUID -PassThru |
         Add-Member -Name HasElementType -MemberType 'NoteProperty' -Value $t.HasElementType -PassThru |
         Add-Member -Name IsAbstract -MemberType 'NoteProperty' -Value $t.IsAbstract -PassThru |
         Add-Member -Name IsAnsiClass -MemberType 'NoteProperty' -Value $t.IsAnsiClass -PassThru |
         Add-Member -Name IsArray -MemberType 'NoteProperty' -Value $t.IsArray -PassThru |
         Add-Member -Name IsAutoClass -MemberType 'NoteProperty' -Value $t.IsAutoClass -PassThru |
         Add-Member -Name IsAutoLayout -MemberType 'NoteProperty' -Value $t.IsAutoLayout -PassThru |
         Add-Member -Name IsByRef -MemberType 'NoteProperty' -Value $t.IsByRef -PassThru |
         Add-Member -Name IsClass -MemberType 'NoteProperty' -Value $t.IsClass -PassThru |
         Add-Member -Name IsCOMObject -MemberType 'NoteProperty' -Value $t.IsCOMObject -PassThru |
         Add-Member -Name IsContextful -MemberType 'NoteProperty' -Value $t.IsContextful -PassThru |
         Add-Member -Name IsEnum -MemberType 'NoteProperty' -Value $t.IsEnum -PassThru |
         Add-Member -Name IsExplicitLayout -MemberType 'NoteProperty' -Value $t.IsExplicitLayout -PassThru |
         Add-Member -Name IsGenericParameter -MemberType 'NoteProperty' -Value $t.IsGenericParameter -PassThru |
         Add-Member -Name IsGenericType -MemberType 'NoteProperty' -Value $t.IsGenericType -PassThru |
         Add-Member -Name IsGenericTypeDefinition -MemberType 'NoteProperty' -Value $t.IsGenericTypeDefinition -PassThru |
         Add-Member -Name IsImport -MemberType 'NoteProperty' -Value $t.IsImport -PassThru |
         Add-Member -Name IsInterface -MemberType 'NoteProperty' -Value $t.IsInterface -PassThru |
         Add-Member -Name IsLayoutSequential -MemberType 'NoteProperty' -Value $t.IsLayoutSequential -PassThru |
         Add-Member -Name IsMarshalByRef -MemberType 'NoteProperty' -Value $t.IsMarshalByRef -PassThru |
         Add-Member -Name IsNested -MemberType 'NoteProperty' -Value $t.IsNested -PassThru |
         Add-Member -Name IsNestedAssembly -MemberType 'NoteProperty' -Value $t.IsNestedAssembly -PassThru |
         Add-Member -Name IsNestedFamANDAssem -MemberType 'NoteProperty' -Value $t.IsNestedFamANDAssem -PassThru |
         Add-Member -Name IsNestedFamily -MemberType 'NoteProperty' -Value $t.IsNestedFamily -PassThru |
         Add-Member -Name IsNestedFamORAssem -MemberType 'NoteProperty' -Value $t.IsNestedFamORAssem -PassThru |
         Add-Member -Name IsNestedPrivate -MemberType 'NoteProperty' -Value $t.IsNestedPrivate -PassThru |
         Add-Member -Name IsNestedPublic -MemberType 'NoteProperty' -Value $t.IsNestedPublic -PassThru |
         Add-Member -Name IsNotPublic -MemberType 'NoteProperty' -Value $t.IsNotPublic -PassThru |
         Add-Member -Name IsPointer -MemberType 'NoteProperty' -Value $t.IsPointer -PassThru |
         Add-Member -Name IsPrimitive -MemberType 'NoteProperty' -Value $t.IsPrimitive -PassThru |
         Add-Member -Name IsPublic -MemberType 'NoteProperty' -Value $t.IsPublic -PassThru |
         Add-Member -Name IsSealed -MemberType 'NoteProperty' -Value $t.IsSealed -PassThru |
         Add-Member -Name IsSerializable -MemberType 'NoteProperty' -Value $t.IsSerializable -PassThru |
         Add-Member -Name IsSpecialName -MemberType 'NoteProperty' -Value $t.IsSpecialName -PassThru |
         Add-Member -Name IsUnicodeClass -MemberType 'NoteProperty' -Value $t.IsUnicodeClass -PassThru |
         Add-Member -Name IsValueType -MemberType 'NoteProperty' -Value $t.IsValueType -PassThru |
         Add-Member -Name IsVisible -MemberType 'NoteProperty' -Value $t.IsVisible -PassThru |
         Add-Member -Name LastAccessTime -MemberType 'NoteProperty' -Value $f.LastAccessTime -PassThru |
         Add-Member -Name LastAccessTimeUtc -MemberType 'NoteProperty' -Value $f.LastAccessTimeUtc -PassThru |
         Add-Member -Name LastWriteTime -MemberType 'NoteProperty' -Value $f.LastWriteTime -PassThru |
         Add-Member -Name LastWriteTimeUtc -MemberType 'NoteProperty' -Value $f.LastWriteTimeUtc -PassThru |
         Add-Member -Name MemberType -MemberType 'NoteProperty' -Value $t.MemberType -PassThru |
         Add-Member -Name MetadataToken -MemberType 'NoteProperty' -Value $t.MetadataToken -PassThru |
         Add-Member -Name Module -MemberType 'NoteProperty' -Value $t.Module -PassThru |
         Add-Member -Name Name -MemberType 'NoteProperty' -Value $t.Name -PassThru |
         Add-Member -Name Namespace -MemberType 'NoteProperty' -Value $t.Namespace -PassThru |
         Add-Member -Name Parent -MemberType 'NoteProperty' -Value $f.Parent -PassThru |
         Add-Member -Name ReflectedType -MemberType 'NoteProperty' -Value $t.ReflectedType -PassThru |
         Add-Member -Name Root -MemberType 'NoteProperty' -Value $f.Root -PassThru |
         Add-Member -Name StructLayoutAttribute -MemberType 'NoteProperty' -Value $t.StructLayoutAttribute -PassThru |
         Add-Member -Name TypeHandle -MemberType 'NoteProperty' -Value $t.TypeHandle -PassThru |
         Add-Member -Name TypeInitializer -MemberType 'NoteProperty' -Value $t.TypeInitializer -PassThru |
         Add-Member -Name UnderlyingSystemType -MemberType 'NoteProperty' -Value $t.UnderlyingSystemType -PassThru |
         Add-Member -Name PSChildName -MemberType 'NoteProperty' -Value $f.PSChildName -PassThru |
         Add-Member -Name PSDrive -MemberType 'NoteProperty' -Value $f.PSDrive -PassThru |
         Add-Member -Name PSIsContainer -MemberType 'NoteProperty' -Value $f.PSIsContainer -PassThru |
         Add-Member -Name PSParentPath -MemberType 'NoteProperty' -Value $f.PSParentPath -PassThru |
         Add-Member -Name PSPath -MemberType 'NoteProperty' -Value $f.PSPath -PassThru |
         Add-Member -Name PSProvider -MemberType 'NoteProperty' -Value $f.PSProvider -PassThru |
         Add-Member -Name BaseName -MemberType 'NoteProperty' -Value $f.BaseName -PassThru |
         Add-Member -Name Clone -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name CompareTo -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name Contains -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name CopyTo -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name Create -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name CreateObjRef -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name CreateSubdirectory -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name Delete -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name EndsWith -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name FindInterfaces -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name FindMembers -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetAccessControl -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetArrayRank -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetConstructor -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetConstructors -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetCustomAttributes -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetDefaultMembers -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetDirectories -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetElementType -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetEnumerator -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetEvent -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetEvents -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetField -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetFields -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetFiles -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetFileSystemInfos -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetGenericArguments -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetGenericParameterConstraints -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetGenericTypeDefinition -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetInterface -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetInterfaceMap -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetInterfaces -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetLifetimeService -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetMember -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetMembers -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetMethod -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetMethods -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetNestedType -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetNestedTypes -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetObjectData -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetProperties -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetProperty -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name GetTypeCode -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name IndexOf -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name IndexOfAny -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name InitializeLifetimeService -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name Insert -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name InvokeMember -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name IsAssignableFrom -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name IsDefined -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name IsInstanceOfType -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name IsNormalized -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name IsSubclassOf -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name LastIndexOf -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name LastIndexOfAny -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name MakeArrayType -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name MakeByRefType -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name MakeGenericType -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name MakePointerType -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name MoveTo -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name Normalize -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name PadLeft -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name PadRight -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name Refresh -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name Remove -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name Replace -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name SetAccessControl -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name Split -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name StartsWith -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name Substring -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name ToCharArray -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name ToLower -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name ToLowerInvariant -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name ToUpper -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name ToUpperInvariant -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name Trim -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name TrimEnd -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name TrimStart -MemberType 'ScriptMethod' -Value {} -PassThru |
         Add-Member -Name Chars -MemberType 'NoteProperty' -Value $s.Chars -PassThru
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
         $global:_dummy = gcm Add-Content
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
         $global:_dummy = Get-EventLog Windows` PowerShell -Newest 1
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
         $global:_dummy = Get-Item .
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
         Add-Member -InputObject (New-Object PSObject) -Name Action -MemberType 'NoteProperty' -Value '' -PassThru |
         Add-Member -Name Command -MemberType 'NoteProperty' -Value '' -PassThru |
         Add-Member -Name Enabled -MemberType 'NoteProperty' -Value '' -PassThru |
         Add-Member -Name HitCount -MemberType 'NoteProperty' -Value '' -PassThru |
         Add-Member -Name Id -MemberType 'NoteProperty' -Value '' -PassThru |
         Add-Member -Name Script -MemberType 'NoteProperty' -Value '' -PassThru
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
         $global:_dummy = (iex $str)[0]
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
 return;
 }
`

