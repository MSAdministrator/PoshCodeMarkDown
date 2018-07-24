---
Author: m skourlatov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6890
Published Date: 2017-05-09t23
Archived Date: 2017-05-13t18
---

# start-foldersync - 

## Description

small script to strongly syncronyze folder with ‘original’.

## Comments



## Usage



## TODO



## class

`start-foldersync`

## Code

`#
 #
 Add-Type -TypeDefinition @"
 
 public struct HashContainer : System.IComparable
 {
 	public ulong High;
 	public ulong Low;
 
 	public HashContainer(byte[] Buffer)
 	{
 		System.Array.Reverse(Buffer);
 		this.Low = System.BitConverter.ToUInt64(Buffer, 0);
 		this.High = System.BitConverter.ToUInt64(Buffer, 8);
 	}
 
 	public int CompareTo(object obj)
 	{
 		if (obj == null) { return 1; }
 		var other = (HashContainer)obj;
 
 		int compare = this.High.CompareTo(other.High);
 		if (compare != 0) return compare;
 		return this.Low.CompareTo(other.Low);
 	}
 
 	public override string ToString()
 	{
 		return string.Format("{0:X}{1:X}", this.High, this.Low);
 	}
 
 }
 
 public class FIComparable : System.IComparable
 {
 	public System.IO.FileInfo Info;
 
 	private bool hasHash;
 	private HashContainer hash;
 
 	public FIComparable(System.IO.FileInfo info)
 	{
 		this.Info = info;
 		this.hasHash = false;
 	}
 
 	public HashContainer GetHash()
 	{
 		if (this.hasHash) { return this.hash; }
 
 		var crypto = new System.Security.Cryptography.MD5CryptoServiceProvider();
 		var stream = new System.IO.FileStream(
 			this.Info.FullName,
 			System.IO.FileMode.Open,
 			System.IO.FileAccess.Read,
 			System.IO.FileShare.None);
 		var buf = crypto.ComputeHash(stream);
 		stream.Close();
 
 		this.hash = new HashContainer(buf);
 		this.hasHash = true;
 		return this.hash;
 	}
 
 	public int CompareTo(object obj)
 	{
 		if (obj == null) { return 1; }
 		var other = obj as FIComparable;
 
 		if (null != other)
 		{
 			int compare = this.Info.Name.CompareTo(other.Info.Name);
 			if (compare != 0) { return compare; }
 			compare = this.Info.Length.CompareTo(other.Info.Length);
 			if (compare != 0) { return compare; }
 			return this.GetHash().CompareTo(other.GetHash());
 		}
 		else { throw new System.ArgumentException(); }
 	}
 	
 	public override string ToString() { return this.Info.Name; }
 }
 "@
 
 Function Start-FolderSync
 {
 	Param
 	(
 		[parameter(Mandatory, Position = 0)]
 		[ValidateScript({ Test-Path -Path $_ -PathType 'Container' })]
 		[string]$Reference,
 		[parameter(Mandatory, Position = 1)]
 		[string]$Synchronized
 	)
 
 	function Get-Comparable($list)
 	{
 		$sync = New-Object 'System.Collections.Generic.List[FIComparable]'
 		foreach ($item in $list) { $sync.Add((New-Object FIComparable $item)) }
 		return $sync
 	}
 
 	$gotcha = Get-ChildItem -LiteralPath $Reference
 	$fileList = New-Object 'System.Collections.Generic.List[System.IO.FileInfo]'
 	$copyList = New-Object 'System.Collections.Generic.List[System.IO.FileInfo]'
 	$dirList = New-Object 'System.Collections.Generic.List[System.IO.DirectoryInfo]'
 
 	foreach ($item in $gotcha)
 	{
 		if (($item.Attributes -band 16) -eq 16)
 			{ $dirList.Add($item) }
 		else
 			{ $fileList.Add($item) }
 	}
 
 	$ref = Get-Comparable $fileList
 
 	$syncDir = [System.IO.Directory]::CreateDirectory($Synchronized)
 	if ($syncDir -eq $null) { return $false }
 
 	$syncFileList = Get-ChildItem -LiteralPath $syncDir.FullName -File
 	$sync = Get-Comparable $syncFileList
 	if ($sync -eq $null -or $ref -eq $null)
 	{
 		$copyList = $fileList
 	}
 	else
 	{
 		$compasion = Compare-Object -ReferenceObject $ref -DifferenceObject $sync
 		foreach ($item in $compasion)
 		{
 			if ($item.SideIndicator -eq '<=')
 			{
 				$copyList.Add($item.InputObject.Info)
 			}
 		}
 	}
 
 	foreach ($item in $dirList)
 	{
 		Start-FolderSync -Reference $item.FullName -Synchronized $($syncDir.FullName + '\' + $item.Name)
 	}
 	
 	foreach ($item in $copyList)
 	{
 		Copy-Item -Path $item.FullName -Destination $syncDir.FullName -Verbose
 	}
 }
`

