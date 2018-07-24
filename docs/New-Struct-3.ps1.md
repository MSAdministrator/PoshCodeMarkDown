---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3146
Published Date: 2012-01-05t20
Archived Date: 2016-03-03t14
---

# new-struct 3 - 

## Description

a code-generating and emitting magic function for creating type-safe struct classes for use in powershell!

## Comments



## Usage



## TODO



## function

`new-struct`

## Code

`#
 #
 function New-Struct {
 #.Synopsis
 #.Description
 #.Example
 #
 #.Example
 #
 #
 [CmdletBinding(DefaultParameterSetName="Multiple")]
 param(
     [ValidateScript({
         if($_ -notmatch '^[a-z][a-z1-9_]*$') {
             throw "'$_' is invalid. A valid name identifier must start with a letter, and contain only alpha-numeric or the underscore (_)."
         }
         return $true             
     })]
     [Parameter(Position=0, Mandatory=$true, ValueFromPipelineByPropertyName=$true, ParameterSetName = "Single")]
     [string]$Name
 ,
     [Parameter(Position=1, Mandatory=$true, ValueFromPipelineByPropertyName=$true, ParameterSetName = "Single")]
     [ScriptBlock]$Property
 ,
     [Parameter(Position=0, Mandatory=$true, ParameterSetName = "Multiple")]
     [HashTable]$Types
 ,
     [Alias("CTorFunction","ConstructorFunction")]
     [Switch]$CreateConstructorFunction
 ,
     [Switch]$PassThru
 )
 begin {
     if($PSCmdlet.ParameterSetName -eq "Multiple") {
         $Structs = foreach($key in $Types.Keys) {
             New-Object PSObject -Property @{Name=$key;Property=$Types.$key}
         }
         Write-Verbose ($Structs | Out-String)
         $Structs | New-Struct -Passthru:$Passthru -CreateConstructorFunction:$CreateConstructorFunction
     } else {
         $code = "using System;`nusing System.Collections;`nusing System.Management.Automation;`n"
         $function = ""
     }
 }
 process {
 if($PSCmdlet.ParameterSetName -ne "Multiple") {
 $parserrors = $null
 $tokens = [System.Management.Automation.PSParser]::Tokenize( $Property, [ref]$parserrors ) | Where-Object { "Newline","StatementSeparator" -notcontains $_.Type }
 
 $Name = $Name.ToUpper()[0] + $Name.SubString(1)
 $ctor = @()
 $setr = @()
 $prop = @()
 $parm = @()
 $cast = @()
 $hash = @()
 $2Str = @()
 
 $(while($tokens.Count -gt 0) {
     $typeToken,$varToken,$tokens = $tokens
     if($typeToken.Type -ne "Type") {
         throw "Syntax error on line $($typeToken.StartLine) Column $($typeToken.Start). Missing Type. The Struct Properties block must contain only statements of the form: [Type]`$Name, see Get-Help New-Struct -Parameter Properties.`n$($typeToken | Out-String)"
     } elseif($varToken.Type -ne "Variable") {
         throw "Syntax error on line $($varToken.StartLine) Column $($varToken.Start). Missing Name. The Struct Properties block must contain only statements of the form: [Type]`$Name, see Get-Help New-Struct -Parameter Properties.`n$($typeToken | Out-String)"
     }
 
     $varName = $varToken.Content.ToUpper()[0] + $varToken.Content.SubString(1)
     $varNameLower = $varName.ToLower()[0] + $varName.SubString(1)
     try {
         Write-Verbose "TypeToken: $($typeToken.Content) $varName"
         if($PSVersionTable.PSVersion.Major -lt 3) {
             $typeName = Invoke-Expression "[$($typeToken.Content)].FullName"
         } else {
             $typeName = Invoke-Expression "$($typeToken.Content).FullName"
         }            
     } catch {
         if($PSVersionTable.PSVersion.Major -lt 3) {
             $typeName = $typeToken.Content
         } else {
             $typeName = $typeToken.Content -replace '\[(.*)\]','$1'
         }
     }
     Write-Verbose "Type Name: $typeName $varName"
     
     $prop += '   public {0} {1};' -f $typeName,$varName
     $setr += '      {0} = {1};' -f $varName,$varNameLower
     $ctor += '{0} {1}' -f $typeName,$varNameLower
     $cast += '      if(input.Properties["{0}"] != null){{ output.{0} = ({1})input.Properties["{0}"].Value; }}' -f $varName,$typeName
     $hash += '      if(hash.ContainsKey("{0}")){{ output.{0} = ({1})hash["{0}"]; }}' -f $varName,$typeName
     $2Str += '"{0} = [{1}]\"" + {0}.ToString() + "\""' -f $varName, $typeName
     if($CreateConstructorFunction) {
         $parm += '[{0}]${1}' -f $typeName,$varName
     }
 })
 
 $code += @"
 public struct $Name {
 $($prop -join "`n")
    public $Name ($( $ctor -join ","))
    {
 $($setr -join "`n")
    }
    public static implicit operator $Name(Hashtable hash)
    {
       $Name output = new $Name();
 $($hash -join "`n")
       return output;
    }
    public static implicit operator $Name(PSObject input)
    {
       $Name output = new $Name();
 $($cast -join "`n")
       return output;
    }
    
    public override string ToString()
    {
       return "@{" + $($2Str -join ' + "; " + ') + "}";
    }
 }
 
 "@
 
 if($CreateConstructorFunction) {
 $function += @"
 Function global:New-$Name {
 [CmdletBinding()]
 param(
 $( $parm -join ",`n" )
 )
 New-Object $Name -Property `$PSBoundParameters
 }
 
 "@
 }
 
 }}
 end {
 if($PSCmdlet.ParameterSetName -ne "Multiple") {
     Write-Verbose "PowerShell Code:`n$function"
 
     Add-Type -TypeDefinition $code -PassThru:$Passthru -ErrorAction Stop
     if($CreateConstructorFunction) {
         Invoke-Expression $function
     }
 }}}
`

