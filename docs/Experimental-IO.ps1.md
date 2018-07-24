---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 2.0
Encoding: ascii
License: cc0
PoshCode ID: 2488
Published Date: 2011-02-03t22
Archived Date: 2017-02-26t16
---

# experimental.io - 

## Description

a simple implemenation of the experimental.io longpath library from the microsoft base class library project as a module.

## Comments



## Usage



## TODO



## module

`where-wildcard`

## Code

`#
 #
 if(!("Microsoft.Experimental.IO.LongPathDirectory" -as [type])) {
    Add-Type -Path $PSScriptRoot\Microsoft.Experimental.IO.dll
 }
 
 Add-Type -TypeDefinition @"
 using System;
 using System.ComponentModel;
 using System.Management.Automation;
 using System.Collections.ObjectModel;
 
 namespace Huddled.Experimental.IO {
    [AttributeUsage(AttributeTargets.Field | AttributeTargets.Property)]
    public class TransformAttribute : ArgumentTransformationAttribute {
       private ScriptBlock _scriptblock;
       private string _noOutputMessage = "Transform Script had no output.";
 
       public override string ToString() {
          return string.Format("[Transform(Script='{{{0}}}')]", Script);
       }
 
       public override Object Transform( EngineIntrinsics engine, Object inputData) {
          try {
             Collection<PSObject> output =
                engine.InvokeCommand.InvokeScript( engine.SessionState, Script, inputData );
             
             if(output.Count > 1) {
                Object[] transformed = new Object[output.Count];
                for(int i =0; i < output.Count;i++) {
                   transformed[i] = output[i].BaseObject;
                }
                return transformed;
             } else if(output.Count == 1) {
                return output[0].BaseObject;
             } else {
                throw new ArgumentTransformationMetadataException(NoOutputMessage);
             }
          } catch (ArgumentTransformationMetadataException) {
             throw;
          } catch (Exception e) {
             throw new ArgumentTransformationMetadataException(string.Format("Transform Script threw an exception ('{0}'). See `$Error[0].Exception.InnerException.InnerException for more details.",e.Message), e);
          }
       }
       
       public TransformAttribute() {
          this.Script = ScriptBlock.Create("{`$args}");
       }
       
       public TransformAttribute( ScriptBlock Script ) {
          this.Script = Script;
       }
 
       public ScriptBlock Script {
          get { return _scriptblock; }
          set { _scriptblock = value; }
       }
       
       public string NoOutputMessage {
          get { return _noOutputMessage; }
          set { _noOutputMessage = value; }
       }  
    }
 }
 "@ 
 
 function Where-Wildcard {
 param(
    [Parameter(ValueFromPipeline=$true)]
    [string[]]$InputObject
 ,
    [Parameter(Position=0)]
    [string[]]$Like = "*"
 ,
    [Parameter(Position=1)]
    [string[]]$NotLike = $null
 )
 process {
    foreach($item in $InputObject) {
       $filename = Split-Path $item -Leaf
       $passthru = $false
       
       foreach($pattern in $Like) {
          $passthru = $passthru -or $filename -like $pattern
       }
       foreach($pattern in $NotLike) {
          $passthru = $passthru -and !($filename -like $pattern)
       }
       if($passthru) { 
          Write-Output $item
       }
    }
 }
 }
 
 
 function Get-LongPath {
 [CmdletBinding(DefaultParameterSetName="AllItems")]
 param(
    [Parameter(Position=0,ValueFromPipelineByPropertyName=$true,ValueFromPipeline=$true)]
    [Alias("FullName")]   
    [string[]]$LiteralPath = $pwd
 ,
    [string[]]$Include = "*"
 ,
    [string[]]$Exclude = $null
 ,
    [Parameter(ParameterSetName="DirectoriesOnly")]
    [Alias("od","do")]
    [switch]$DirectoriesOnly
 , 
    [Parameter(ParameterSetName="FilesOnly")]
    [Alias("of","fo")]
    [switch]$FilesOnly
 ,
    [switch]$Recurse
 ,
    [switch]$Indent
 ,
    [switch]$OrderDirectoriesFirst
 )
 begin {
    if($Recurse -and $Indent -and (Test-Path variable:script:pad)) {
       $script:pad += "  "
    } else {
       $script:pad = ""
    }
    $null = $PSBoundParameters.Remove("LiteralPath")
    if($PSCmdlet.ParameterSetName -eq "FilesOnly") {
       Write-Verbose $LiteralPath
    }
    
 }
 process {
    foreach($Path in $LiteralPath) {
       if(![Microsoft.Experimental.IO.LongPathDirectory]::Exists($Path)){
          $Include = Split-Path $Path -Leaf
          $Path = Split-Path $Path 
       }
    
       switch($PSCmdlet.ParameterSetName) {
          "FilesOnly" {
             if($Recurse) {
                [Microsoft.Experimental.IO.LongPathDirectory]::EnumerateFileSystemEntries( $Path ) | ForEach-Object{ 
                   if( [Microsoft.Experimental.IO.LongPathDirectory]::Exists( $_ ) ) {
                      Get-LongPath $_ @PSBoundParameters
                   } else {
                      Where-Wildcard -InputObject $_ -Like $Include -NotLike $Exclude | ForEach-Object{ $script:pad + $_ }
                   }
                }
             } else {
                [Microsoft.Experimental.IO.LongPathDirectory]::EnumerateFiles( $Path ) | Where-Wildcard -like $Include -notLike $Exclude
             }
          }
          "DirectoriesOnly" {
             if($Recurse) {
                [Microsoft.Experimental.IO.LongPathDirectory]::EnumerateDirectories( $Path ) | ForEach-Object{ 
                   Where-Wildcard -InputObject $_ -Like $Include -NotLike $Exclude | ForEach-Object{ $script:pad + $_ + "\" }
                   if($recurse) {
                      Get-LongPath $_ @PSBoundParameters
                   }
                }
             } else {
                [Microsoft.Experimental.IO.LongPathDirectory]::EnumerateDirectories( $Path ) | Where-Wildcard -Like $Include -notLike $Exclude
             }
          }
          "AllItems" {
             if($OrderDirectoriesFirst) {
                [Microsoft.Experimental.IO.LongPathDirectory]::EnumerateDirectories( $Path ) | Where-Wildcard -Like $Include -NotLike $Exclude | ForEach-Object{ $script:pad + $_ + "\" }
                [Microsoft.Experimental.IO.LongPathDirectory]::EnumerateFiles( $Path ) | Where-Wildcard -like $Include -notLike $Exclude
             } else {
                if($recurse) {
                   [Microsoft.Experimental.IO.LongPathDirectory]::EnumerateFileSystemEntries( $Path ) | ForEach-Object{ 
                      if( [Microsoft.Experimental.IO.LongPathDirectory]::Exists( $_ ) ) {
                         Where-Wildcard -InputObject $_ -Like $Include -NotLike $Exclude | ForEach-Object{ $script:pad + $_ + "\" }
                         Get-LongPath $_ @PSBoundParameters
                      } else {
                         Where-Wildcard -InputObject $_ -Like $Include -NotLike $Exclude | ForEach-Object{ $script:pad + $_ }
                      }
                   }
                } else { 
                   [Microsoft.Experimental.IO.LongPathDirectory]::EnumerateFileSystemEntries( $Path ) | 
                      Where-Wildcard -like $Include -notLike $Exclude |
                      ForEach-Object {
                         if( [Microsoft.Experimental.IO.LongPathDirectory]::Exists( $_ ) ) {
                            $script:pad + $_ + "\"
                         } else {
                            $script:pad + $_
                         }
                      }
                }
             }
          }
       }
    }
 }
 end {
    if($Indent) {
       if($script:pad.Length -gt 0) {
          $script:pad = $script:pad.SubString(0, $script:pad.Length - 2)
       } else {
          remove-item variable:script:pad -EA 0
       }
    }
 }
 }
 
 $ExperimentalFormatColums = 4
 
 
 function Format-Color {
 param(
    [Parameter(ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
    [Alias("FullName")]
    [string[]]$InputObject
 ,
    [ConsoleColor]$Directories = 'Cyan'
 ,
    [ConsoleColor]$Executable = 'Green'
 ,
    [int]$Columns = $ExperimentalFormatColums
 )
 begin {
    $NoNewLine = $columns -gt 0
    $Count = 0
 }
 
 process {
    foreach($item in $InputObject) {
       $Count = $Count + 1
       $filename = Split-Path $item -Leaf
 
       $output = $item
       if($NoNewLine) {
          $width = $Host.UI.RawUI.BufferSize.Width / $Columns
          if( [Microsoft.Experimental.IO.LongPathDirectory]::Exists( $item ) ) {
             $output = "$filename\".PadRight( $width ).SubString(0,$width-1).PadRight( $width )
          } else {
             $output = $filename.PadRight( $width ).SubString(0,$width-1).PadRight( $width )
          }
       }
       
       if( [Microsoft.Experimental.IO.LongPathDirectory]::Exists( $item ) ) {
          Write-Host $output -Fore $Directories -NoNewLine:$NoNewLine
       } 
       elseif( Where-Wildcard -InputObject $item -Like "*.exe","*.cmd","*.bat","*.ps1") {
          Write-Host $output -Fore $Executable -NoNewLine:$NoNewLine
       }
       else {
          Write-Host $output -NoNewLine:$NoNewLine
       }
       
    }
 }
 end {
    if($NoNewLine -and (($Count % $Columns) -ne 0)) {
       Write-Host
    }
 }
 }
 
 
 function Copy-LongPath {
 param(
    [Parameter(ValueFromPipeline=$true)]
    [string[]]$LiteralPath
 ,
    [Parameter(ValueFromPipeline=$true)]
    [string]$Destination
 ,
    [switch]$Force
 )
 process {
    foreach($item in $LiteralPath) {
       if( [Microsoft.Experimental.IO.LongPathDirectory]::Exists($Destination) ) {
          $target = Join-Path $Destination (Split-Path $item -Leaf)
       } else {
          $target = $Destination
       }
       
       [Microsoft.Experimental.IO.LongPathFile]::Copy($item, $target, $force)
    }
 }
 }
 
 
 function Move-LongPath {
 param(
    [Parameter(ValueFromPipeline=$true)]
    [string[]]$LiteralPath
 ,
    [Parameter(ValueFromPipeline=$true)]
    [string]$Destination
 ,
    [switch]$Force
 )
 process {
    foreach($item in $LiteralPath) {
       if( [Microsoft.Experimental.IO.LongPathDirectory]::Exists($Destination) ) {
          $target = Join-Path $Destination (Split-Path $item -Leaf)
       } else {
          $target = $Destination
       }
       if([Microsoft.Experimental.IO.LongPathFile]::Exists($target) -and $Force) {
          [Microsoft.Experimental.IO.LongPathFile]::Delete($target)
       }
       [Microsoft.Experimental.IO.LongPathFile]::Copy($item, $target)
    }
 }
 }
 
 function Remove-LongPath {
 param(
    [Parameter(ValueFromPipeline=$true)]
    [string[]]$LiteralPath
 )
 process {
    foreach($item in $LiteralPath) {
       [Microsoft.Experimental.IO.LongPathFile]::Delete($item)
    }
 }
 }
 
 
 function Get-ContentLongPath {
 param(
    [Parameter(ValueFromPipeline=$true,Position=0)]
    [string[]]$LiteralPath
 ,
    [switch]$All
 )
 process {
    foreach($path in $LiteralPath) {
       $stream = [Microsoft.Experimental.IO.LongPathFile]::Open( $path, "Open", "Read" )
       $reader = New-Object System.IO.StreamReader $stream, $true
       if($All) {
          $reader.ReadToEnd()
       } else {
          while(!$reader.EndOfStream) {
             $reader.ReadLine()
          }
       }
       $reader.Close()
       $stream.Close()
    }
 }
 }
 
 function Set-ContentLongPath {
 param(
    [Parameter(Position=0)]
    [string[]]$LiteralPath
 ,
    [Parameter(Position=1, ValueFromPipeline=$true)]
    [string[]]$Value
 ,
    [System.Text.Encoding]
    [Huddled.Experimental.IO.Transform({
       param([string]$Encoding)
          $coding = [System.Text.Encoding]::GetEncodings() | Where-Object { $_.Name -like $Encoding } | Select -First 1
          if(!$coding) {
             $coding = [System.Text.Encoding]::GetEncodings() | Where-Object { $_.DisplayName -like $Encoding } | Select -First 1
          }
          [Text.Encoding]::GetEncoding( $coding.CodePage ) 
    })]
    $Encoding = "UTF-8"
 )
 begin {
    $streams = @()
    $writers = @()
    
    foreach($item in $LiteralPath) {
       $stream = [Microsoft.Experimental.IO.LongPathFile]::Open( $LiteralPath, "OpenOrCreate", "Write" )
       $streams += $stream
       $writers += New-Object System.IO.StreamWriter $stream, $encoding
    }
 }
 process {
    foreach($writer in $writers) {
       foreach($v in $Value) {
          $writer.WriteLine( $v )
       }
    }
 }
 end {
    foreach($writer in $writers) { $writer.Close() }
    foreach($stream in $streams) { $stream.Close() }
 }
 }
 
 
 
 New-Alias fco  Format-Color        -ErrorAction SilentlyContinue
 New-Alias glp  Get-LongPath        -ErrorAction SilentlyContinue
 New-Alias cplp Copy-LongPath       -ErrorAction SilentlyContinue
 New-Alias mvlp Move-LongPath       -ErrorAction SilentlyContinue
 New-Alias rmlp Remove-LongPath     -ErrorAction SilentlyContinue
 New-Alias gclp Get-ContentLongPath -ErrorAction SilentlyContinue
 New-Alias sclp Set-ContentLongPath -ErrorAction SilentlyContinue
 
 Export-ModuleMember -Alias * -Function Copy-LongPath, Format-Color, Get-ContentLongPath, Get-LongPath, Move-LongPath, Remove-LongPath, Set-ContentLongPath
`

