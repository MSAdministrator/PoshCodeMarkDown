---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3386
Published Date: 2012-04-28t21
Archived Date: 2015-05-01t23
---

# write-indented - 

## Description

wrappers for write-host and write-verbose that support indenting, including automatic indenting based on stack depth.

## Comments



## Usage



## TODO



## function

`write-host`

## Code

`#
 #
 function Write-Host {
 #.Synopsis
 #.Description
 #
 #.Example
 #
 #.Example
 #
 #.Example
 #
 #.Example
 #
 #.Inputs
 #.Outputs
 [CmdletBinding(HelpUri='http://go.microsoft.com/fwlink/?LinkID=113426', RemotingCapability='None')]
 param(
    [Parameter(Position=0, ValueFromPipeline=$true, ValueFromRemainingArguments=$true)]
    [System.Object[]]
    ${Object},
 
    [switch]
    ${NoNewline},
 
    [System.Object]
    ${Separator},
 
    [System.ConsoleColor]
    ${ForegroundColor},
 
    [System.ConsoleColor]
    ${BackgroundColor},
 
    [Switch]
    $AutoIndent = $(if($Global:WriteHostAutoIndent){$Global:WriteHostAutoIndent}else{$False}),
    
    [Int]
    $PadIndent = $(if($Global:WriteHostPadIndent){$Global:WriteHostPadIndent}else{0}),
 
    [Int]
    $IndentSize = $(if($Global:WriteHostIndentSize){$Global:WriteHostIndentSize}else{2})
 )
 begin
 {
    function Get-ScopeDepth { 
       $depth = 0
       do { $null = Get-Variable PID -Scope (++$depth) } while($?)
       return $depth - 3
    }
    
    if($PSBoundParameters.ContainsKey("AutoIndent")) { $null = $PSBoundParameters.Remove("AutoIndent") }
    if($PSBoundParameters.ContainsKey("PadIndent"))  { $null = $PSBoundParameters.Remove("PadIndent")  }
    if($PSBoundParameters.ContainsKey("IndentSize")) { $null = $PSBoundParameters.Remove("IndentSize") }
    
    $Indent = $PadIndent
    
    if($AutoIndent) { $Indent += (Get-ScopeDepth) * $IndentSize }
    $Width = $Host.Ui.RawUI.BufferSize.Width - $Indent
 
    if($PSBoundParameters.ContainsKey("Object")) {
       $OFS = $Separator
       $PSBoundParameters["Object"] = $(
          foreach($line in $Object) {
             $line = "$line".Trim("`n").Trim("`r")
             for($start = 0; $start -lt $line.Length; $start += $Width -1) {
                $Count = if($Width -gt ($Line.Length - $start)) { $Line.Length - $start} else { $Width - 1}
                (" " * $Indent) + $line.SubString($start,$Count).Trim()
             }
          }
       ) -join ${Separator}
    }
    
     try {
         $outBuffer = $null
         if ($PSBoundParameters.TryGetValue('OutBuffer', [ref]$outBuffer))
         {
             $PSBoundParameters['OutBuffer'] = 1
         }
         $wrappedCmd = $ExecutionContext.InvokeCommand.GetCommand('Write-Host', [System.Management.Automation.CommandTypes]::Cmdlet)
         $scriptCmd = {& $wrappedCmd @PSBoundParameters }
         $steppablePipeline = $scriptCmd.GetSteppablePipeline($myInvocation.CommandOrigin)
         $steppablePipeline.Begin($PSCmdlet)
     } catch {
         throw
     }
 }
 
 process
 {
     try {
       $OFS = $Separator
       $_ = $(
          foreach($line in $_) {
             $line = "$line".Trim("`n").Trim("`r")
             for($start = 0; $start -lt $line.Length; $start += $Width -1) {
                $Count = if($Width -gt ($Line.Length - $start)) { $Line.Length - $start} else { $Width - 1}
                (" " * $Indent) + $line.SubString($start,$Count).Trim()
             }
          }
       ) -join ${Separator}
       $steppablePipeline.Process($_)
     } catch {
         throw
     }
 }
 
 end
 {
     try {
         $steppablePipeline.End()
     } catch {
         throw
     }
 }
 }
 
 
 function Write-Verbose {
 #.Synopsis
 #.Description
 #.Example
 #
 #.Example
 #
 #.Example
 #
 #.Inputs
 #.Outputs
 [CmdletBinding(HelpUri='http://go.microsoft.com/fwlink/?LinkID=113429', RemotingCapability='None')]
 param(
    [Parameter(Position=0, ValueFromPipeline=$true, ValueFromRemainingArguments=$true)]
    [System.Object[]]
    ${Message},
 
    [System.Object]
    ${Separator},
 
    [Switch]
    $AutoIndent = $(if($Global:WriteVerboseAutoIndent){$Global:WriteVerboseAutoIndent}else{$False}),
    
    [Int]
    $PadIndent = $(if($Global:WriteVerbosePadIndent){$Global:WriteVerbosePadIndent}else{0}),
 
    [Int]
    $IndentSize = $(if($Global:WriteVerboseIndentSize){$Global:WriteVerboseIndentSize}else{2})
 )
 begin
 {
    function Get-ScopeDepth { 
       $depth = 0
       do { $null = Get-Variable PID -Scope (++$depth) } while($?)
       return $depth - 3
    }
    
    if($PSBoundParameters.ContainsKey("AutoIndent")) { $null = $PSBoundParameters.Remove("AutoIndent") }
    if($PSBoundParameters.ContainsKey("PadIndent"))  { $null = $PSBoundParameters.Remove("PadIndent")  }
    if($PSBoundParameters.ContainsKey("IndentSize")) { $null = $PSBoundParameters.Remove("IndentSize") }
    if($PSBoundParameters.ContainsKey("Separator")) { $null = $PSBoundParameters.Remove("Separator") }
    
    $Indent = $PadIndent
    
    if($AutoIndent) { $Indent += (Get-ScopeDepth) * $IndentSize }
    $Prefix = "VERBOSE: ".Length
    $Width = $Host.Ui.RawUI.BufferSize.Width - $Indent - $Prefix
 
    
    if($PSBoundParameters.ContainsKey("Message")) {
       $OFS = $Separator
       $PSBoundParameters["Message"] = $(
          foreach($line in $Message) {
             $line = "$line".Trim("`n").Trim("`r")
             for($start = 0; $start -lt $line.Length; $start += $Width - 1) {
                $Count = if($Width -gt ($Line.Length - $start)) { $Line.Length - $start} else { $Width - 1}
                if($start) { 
                   (" " * ($Indent + $Prefix)) + $line.SubString($start,$Count).Trim()
                } else {
                   (" " * $Indent) + $line.SubString($start,$Count).Trim()
                }
             }
          }
       ) -join "`n"
    }
    
     try {
         $outBuffer = $null
         if ($PSBoundParameters.TryGetValue('OutBuffer', [ref]$outBuffer))
         {
             $PSBoundParameters['OutBuffer'] = 1
         }
         $wrappedCmd = $ExecutionContext.InvokeCommand.GetCommand('Write-Verbose', [System.Management.Automation.CommandTypes]::Cmdlet)
         $scriptCmd = {& $wrappedCmd @PSBoundParameters }
         $steppablePipeline = $scriptCmd.GetSteppablePipeline($myInvocation.CommandOrigin)
         $steppablePipeline.Begin($PSCmdlet)
     } catch {
         throw
     }
 }
 
 process
 {
     try {
       $OFS = $Separator
       $_ = $(
          foreach($line in $_) {
             $line = "$line".Trim("`n").Trim("`r")
             for($start = 0; $start -lt $line.Length; $start += $Width - 1) {
                $Count = if($Width -gt ($Line.Length - $start)) { $Line.Length - $start} else { $Width - 1}
                if($start) { 
                   (" " * ($Indent + $Prefix)) + $line.SubString($start,$Count).Trim()
                } else {
                   (" " * $Indent) + $line.SubString($start,$Count).Trim()
                }
                
             }
          }
       ) -join "`n"
       $steppablePipeline.Process($_)
     } catch {
         throw
     }
 }
 
 end
 {
     try {
         $steppablePipeline.End()
     } catch {
         throw
     }
 }
 }
`

