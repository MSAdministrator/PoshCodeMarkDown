---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.7
Encoding: ascii
License: cc0
PoshCode ID: 920
Published Date: 2009-03-07t23
Archived Date: 2012-06-27t00
---

# resolve-aliases - 

## Description

resolves aliases and parameter shortcuts in scripts to make them more portable.  now resolves parameter aliases, and resolves ‘?’ to where-object correctly.

## Comments



## Usage



## TODO



## module

`resolve-aliases`

## Code

`#
 #
 ########################################################################################################################
 ########################################################################################################################
 ##
 ########################################################################################################################
 function which {
 PARAM( [string]$command )
    $cmds = @(Get-Command $command -EA "SilentlyContinue")
    if($cmds.Count -gt 1) {
       $cmd = @( $cmds | Where-Object { $_.Name -match "^$([Regex]::Escape($command))" })[0]
    } else {
       $cmd = $cmds[0]
    }
    if(!$cmd) {
       $cmd = @(Get-Command "Get-$command" -EA "SilentlyContinue" | Where-Object { $_.Name -match "^Get-$([Regex]::Escape($command))" })[0]
    }
    if( $cmd.CommandType -eq "Alias" ) {
       $cmd = which $cmd.Definition
    }
    return $cmd
 }
 
 function Resolve-Aliases
 {
    [CmdletBinding(ConfirmImpact="low",DefaultParameterSetName="Files")]
    Param (
       [Parameter(Position=0, Mandatory=$true, ValueFromPipeline=$true, ParameterSetName="Text")]
       [string]$Line
 ,
       [Parameter(Position=0, Mandatory=$true, ValueFromPipeline=$true, ParameterSetName="Files")]
       [Alias("FullName","PSChildName","PSPath")]
       [IO.FileInfo]$File
 ,
       [Parameter(Position=1, ParameterSetName="Files")] 
       [Switch]$InPlace
    )
    BEGIN {
       Write-Debug $PSCmdlet.ParameterSetName
    }
    PROCESS {
       if($PSCmdlet.ParameterSetName -eq "Files") {
          if($File -is [System.IO.FileInfo]){
             $Line = ((Get-Content $File) -join "`n")
          } else {
             throw "We can't resolve a whole folder at once yet" 
          }
       }
 
       $Tokens = [System.Management.Automation.PSParser]::Tokenize($Line,[ref]$null)
       for($t = $Tokens.Count; $t -ge 0; $t--) {
          $token = $Tokens[$t]
          switch($token.Type) {
             "Command" {
                $cmd = which $token.Content
                Write-Debug "Command $($token.Content) => $($cmd.Name)"
                $Line = $Line.Remove( $token.Start, $token.Length ).Insert( $token.Start, $cmd.Name )
                #}
             }
             "CommandParameter" {
                Write-Debug "Parameter $($token.Content)"
                for($c = $t; $c -ge 0; $c--) {
                   if( $Tokens[$c].Type -eq "Command" ) {
                      $cmd = which $Tokens[$c].Content
                      $short = $token.Content -replace "^-?","^"
                      Write-Debug "Parameter $short"
                      $parameters = $cmd.ParameterSets | Select -expand Parameters
                      $param = @($parameters | Where-Object { $_.Name -match $short -or $_.Aliases -match $short} | Select -Expand Name -Unique)
                      if($param.Count -eq 1) {
                         $Line = $Line.Remove( $token.Start, $token.Length ).Insert( $token.Start, "-$($param[0])" )
                      }
                      break
                   }
                }
             }
          }
       }
 
       switch($PSCmdlet.ParameterSetName) {
          "Text" {
             $Line
          }
          "Files" {
             switch($File.GetType()) {
                "System.IO.FileInfo" {
                   if($InPlace) {
                      $Line | Set-Content $File 
                   } else {
                      $Line
                   }
                }
                default { throw "We can't resolve a whole folder at once yet" }
             }
          }
          default { throw "ParameterSet: $($PSCmdlet.ParameterSetName)" }
       }
    }
 }
`

