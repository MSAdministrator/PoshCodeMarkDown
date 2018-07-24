---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 4.5
Encoding: ascii
License: cc0
PoshCode ID: 4041
Published Date: 2016-03-25t20
Archived Date: 2016-03-17t03
---

# reflection module - 

## Description

helpers for working with .net classes

## Comments



## Usage



## TODO



## function

`get-type`

## Code

`#
 #
 #
 
 Add-Type -TypeDefinition @"
 using System;
 using System.ComponentModel;
 using System.Management.Automation;
 using System.Collections.ObjectModel;
 
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
 "@ 
 
 
 
 function Get-Type {
    <#
       .Synopsis
          Gets the types that are currenty loaded in .NET, or gets information about a specific type
       .Description
          Gets information about one or more loaded types, or gets the possible values for an enumerated type or value.    
       .Example
          Get-Type
           
          Gets all loaded types (takes a VERY long time to print out)
       .Example
          Get-Type -Assembly ([PSObject].Assembly)
           
          Gets types from System.Management.Automation
       .Example
          [Threading.Thread]::CurrentThread.ApartmentState | Get-Type
           
          Gets all of the possible values for the ApartmentState property
       .Example
          [Threading.ApartmentState] | Get-Type
           
          Gets all of the possible values for an apartmentstate
    #>
    [CmdletBinding(DefaultParameterSetName="Assembly")]   
    param(
       [Parameter(ValueFromPipeline=$true)]
       [PsObject[]]$Assembly,
 
       [Parameter(Mandatory=$false,Position=0)]
       [SupportsWildCards()]
       [String[]]$TypeName,
 
       [Parameter(Mandatory=$false)]
       [SupportsWildCards()]
       [String[]]$Namespace,
 
       [Parameter(Mandatory=$false)]
       [SupportsWildCards()]
       [String[]]$BaseType,
 
       [Parameter(Mandatory=$false)]
       [SupportsWildCards()]
       [String[]]$Interface,
 
       [Parameter(Mandatory=$false)]
       [SupportsWildCards()]
       [String[]]$Attribute,
 
 
       [Parameter(ParameterSetName="Enum")]
       [PSObject]$Enum, 
 
       [Parameter()][Alias("Private","ShowPrivate")]
       [Switch]$Force
    )
 
    process {
       if($psCmdlet.ParameterSetName -eq 'Enum') {
          if($Enum -is [Enum]) {
             [Enum]::GetValues($enum.GetType())
          } elseif($Enum -is [Type] -and $Enum.IsEnum) {
             [Enum]::GetValues($enum)
          } else {
             throw "Specified Enum is neither an enum value nor an enumerable type"
          }
       }
       else {
          if($Assembly -as [Reflection.Assembly[]]) { 
          } elseif($Assembly -as [String[]]) {
             $Assembly = Get-Assembly $Assembly
          } elseif(!$Assembly) {
             $Assembly = [AppDomain]::CurrentDomain.GetAssemblies()
          }
 
          :asm foreach ($asm in $assembly) {
             Write-Verbose "Testing Types from Assembly: $($asm.Location)"
             if ($asm) { 
                trap {
                   if( $_.Exception.LoaderExceptions -and $_.Exception.LoaderExceptions[0] -is [System.IO.FileNotFoundException] ) {
                      $PSCmdlet.WriteWarning( "Unable to load some types from $($asm.Location), required assemblies were not found. Use -Debug to see more detail")
                      continue asm
                   }
                   Write-Error "Unable to load some types from $($asm.Location). Try with -Debug to see more detail"
                   Write-Debug $( $_.Exception.LoaderExceptions | Out-String )
                   continue asm
                }
                $asm.GetTypes() | Where {
                   ( $Force -or $_.IsPublic ) -AND
                   ( !$Namespace -or $( foreach($n in $Namespace) { $_.Namespace -like $n  } ) ) -AND
                   ( !$TypeName -or $( foreach($n in $TypeName) { $_.Name -like $n -or $_.FullName -like $n } ) -contains $True ) -AND
                   ( !$Attribute -or $( foreach($n in $Attribute) { $_.CustomAttributes | ForEach { $_.AttributeType.Name -like $n -or $_.AttributeType.FullName -like $n } } ) -contains $True ) -AND
                   ( !$BaseType -or $( foreach($n in $BaseType) { $_.BaseType -like $n } ) -contains $True ) -AND
                   ( !$Interface -or @( foreach($n in $Interface) { $_.GetInterfaces() -like $n } ).Count -gt 0 )
                }
             }
          }
       }
    }
 }
 
 function Add-Assembly {
    #.Synopsis
    #.Description
    #.Parameter Path
    #.Parameter Passthru
    #.Parameter Recurse
    [CmdletBinding()]
    param(
       [Parameter(ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true, Position=0)]
       [Alias("PSPath")]
       [string[]]$Path = ".",
 
       [Alias("Types")]
       [Switch]$Passthru,
 
       [Switch]$Recurse
    )
    process {
       foreach($file in Get-ChildItem $Path -Filter *.dll -Recurse:$Recurse) {
          Add-Type -Path $file.FullName -Passthru:$Passthru | Where { $_.IsPublic }
       }
    }
 }
 
 function Get-Assembly {
    <#
    .Synopsis 
       Get a list of assemblies available in the runspace
    .Description
       Returns AssemblyInfo for all the assemblies available in the current AppDomain, optionally filtered by partial name match
    .Parameter Name
       A regex to filter the returned assemblies. This is matched against the .FullName or Location (path) of the assembly.
    #>
    [CmdletBinding()]
    param(
       [Parameter(ValueFromPipeline=$true, Position=0)]
       [string[]]$Name = ''
    )
    process {
       [appdomain]::CurrentDomain.GetAssemblies() | Where {
          $Assembly = $_
          if($Name){ 
             $(
                foreach($n in $Name){
                   if(Resolve-Path $n -ErrorAction 0) {
                      $n = [Regex]::Escape( (Resolve-Path $n).Path )
                   }
                   $Assembly.FullName -match $n -or $Assembly.Location -match $n -or ($Assembly.Location -and (Split-Path $Assembly.Location) -match $n)
                }
             ) -contains $True 
          } else { $true }
       }         
    }
 }
 
 function Update-PSBoundParameters { 
    #.Synopsis
    #.Description
    #.Parameter Name
    #.Parameter Default
    #.Parameter Min
    #.Parameter Max
    #.Parameter PSBoundParameters
    Param(
       [Parameter(Mandatory=$true,  Position=0)]
       [String]$Name,
 
       [Parameter(Mandatory=$false, Position=1)]
       $Default,
 
       [Parameter()]
       $Min,
 
       [Parameter()]
       $Max,
 
       [Parameter(Mandatory=$true, Position=99)]
       $PSBoundParameters=$PSBoundParameters
    )
    end {
       $outBuffer = $null
       if($Default) {
          if (!$PSBoundParameters.TryGetValue($Name, [ref]$outBuffer))
          {
             $PSBoundParameters[$Name] = $Default
          }
       }
       if($Max) {
          if ($PSBoundParameters.TryGetValue($Name, [ref]$outBuffer) -and $outBuffer -gt $Max)
          {
             $PSBoundParameters[$Name] = $Max
          }
       }
       if($Min) {
          if ($PSBoundParameters.TryGetValue($Name, [ref]$outBuffer) -and $outBuffer -lt $Min)
          {
             $PSBoundParameters[$Name] = $Min
          }
       }
       $PSBoundParameters
    }
 }
 
 function Get-Constructor {
    <#
    .Synopsis 
       Returns RuntimeConstructorInfo for the (public) constructor methods of the specified Type.
    .Description
       Get the RuntimeConstructorInfo for a type and add members "Syntax," "SimpleSyntax," and "Definition" to each one containing the syntax information that can use to call that constructor.
    .Parameter Type
       The type to get the constructor for
    .Parameter Force
       Force inclusion of Private and Static constructors which are hidden by default.
    .Parameter NoWarn
       Serves as the replacement for the broken -WarningAction. If specified, no warnings will be written for types without public constructors.
    .Example
       Get-Constructor System.IO.FileInfo
       
       Description
       -----------
       Gets all the information about the single constructor for a FileInfo object. 
    .Example
       Get-Type System.IO.*info mscorlib | Get-Constructor -NoWarn | Select Syntax
       
       Description
       -----------
       Displays the constructor syntax for all of the *Info objects in the System.IO namespace. 
       Using -NoWarn supresses the warning about System.IO.FileSystemInfo not having constructors.
      
    .Example
       $path = $pwd
       $driveName = $pwd.Drive
       $fileName = "$Profile"
       Get-Type System.IO.*info mscorlib | Get-Constructor -NoWarn | ForEach-Object { Invoke-Expression $_.Syntax }
       
       Description
       -----------
       Finds and invokes the constructors for DirectoryInfo, DriveInfo, and FileInfo.
       Note that we pre-set the parameters for the constructors, otherwise they would fail with null arguments, so this example isn't really very practical.
 
 
    #>
    [CmdletBinding()]
    param( 
       [Parameter(Mandatory=$true, ValueFromPipeline=$True, ValueFromPipelineByPropertyName=$true, Position=0)]
       [Alias("ParameterType")]
       [Type]$Type,
       [Switch]$Force,
       [Switch]$NoWarn
    )
    process { 
       $type.GetConstructors() | Where-Object { $Force -or $_.IsPublic -and -not $_.IsStatic } -OutVariable ctor 
       if(!$ctor -and !$NoWarn) { Write-Warning "There are no public constructors for $($type.FullName)" }
    }
 }
 
 function Get-ExtensionMethod {
    <#
       .Synopsis
          Finds Extension Methods which target the specified type
       .Example
          Get-ExtensionMethod String
 
          Finds all extension methods which target strings
    #>
    [CmdletBinding()]
    param(
       [Parameter(Mandatory=$false,Position=0)]
       [SupportsWildCards()]
       [String[]]$TargetTypeName,
 
       [Parameter(Mandatory=$false)]
       [SupportsWildCards()]
       [String[]]$Name = "*",
 
       [Parameter(Mandatory=$false,Position=99)]
       [SupportsWildCards()]
       [String[]]$TypeName = "*"
    )
    process {
       Get-Type -TypeName $TypeName -Attribute ExtensionAttribute | 
          Get-Method -Name $Name -BindingFlags "Static,Public,NonPublic" -Attribute ExtensionAttribute |
          ForEach-Object { 
             $Method = $_
             $ParameterType = $_.GetParameters()[0].ParameterType
 
             ForEach($T in $TargetTypeName) {
                Write-Verbose "Is '$T' a '$ParameterType'?"
                if($ParameterType.Name -like $T -or $ParameterType.FullName -like $T) {
                   Write-Verbose "The name '$T' matches '$ParameterType'"
                   Add-Member -Input $Method -Type NoteProperty -Name ParamBlock -Value (Get-MemberSignature $Method -ParamBlock) -Force
                   Write-Output $Method
                   continue
                }
                
                if($ParameterType.IsGenericType) {
                   $interface = $null
                   if(Test-AssignableToGeneric $T $ParameterType -interface ([ref]$interface)) {
                      Write-Verbose "'$T' is a generic that's assignable to '$ParameterType'"
                      Add-Member -Input $Method -Type NoteProperty -Name Extends -Value $interface.Value -Force
                      Add-Member -Input $Method -Type NoteProperty -Name ParamBlock -Value (Get-MemberSignature $Method -GenericArguments $interface.GetGenericArguments() -ParamBlock) -Force
                      Write-Output $Method
                      continue
                   }
                } else {
                   if($ParameterType.IsAssignableFrom($T)) {
                      Write-Verbose "'$ParameterType' is assignable from '$T'"
                      Add-Member -Input $Method -Type NoteProperty -Name ParamBlock -Value (Get-MemberSignature $Method -ParamBlock) -Force
                      Write-Output $Method
                      continue
                   }     
                }
             }
          }
    }
 }
 
 function Test-AssignableToGeneric { 
    <#
       .Synopsis
          Determine if a specific type can be cast to the given generic type
    #>
    param(
       [Parameter(Position=0, Mandatory = $true)]
       [Type]$type,
 
       [Parameter(ValueFromPipeline=$true, Position=1, Mandatory = $true)]
       [Type]$genericType,
 
       [Switch]$force,
 
       [Parameter(Position=2)]
       [ref]$interface = [ref]$null
    )
 
    process {
       $interfaces = $type.GetInterfaces()
       if($type.IsGenericType -and ($type.GetGenericTypeDefinition().equals($genericType))) {
          return $true
       }
 
       foreach($i in $interfaces) 
       { 
          if($i.IsGenericType -and $i.GetGenericTypeDefinition().Equals($genericType)) {
             $interface.Value = $i
             return $true
          }
          if($i.IsGenericType -and $i.GetGenericTypeDefinition().Equals($genericType.GetGenericTypeDefinition())) {
             $genericTypeArgs = @($genericType.GetGenericArguments())[0]
             if(($genericTypeArgs.IsGenericParameter -and 
                 $genericTypeArgs.BaseType.IsAssignableFrom( @($i.GetGenericArguments())[0] ) ) -or 
                $genericTypeArgs.IsAssignableFrom( @($i.GetGenericArguments())[0] )) {
                
                $interface.Value = $i
                return $true
             }
          }
       }
       if($force -and $genericType -ne $genericType.GetGenericTypeDefinition()) {
          if(Test-AssignableToGeneric $type $genericType.GetGenericTypeDefinition()) {
             return $true
          }
       }
 
       $base = $type.BaseType
       if(!$base) { return $false }
 
       Test-AssignableToGeneric $base $genericType
    }
 }
 
 function Get-Method {
    <#
    .Synopsis 
       Returns MethodInfo for the (public) methods of the specified Type.
    .Description
       Get the MethodInfo for a type and add members "Syntax," "SimpleSyntax," and "Definition" to each one containing the syntax information that can use to call that method.
    .Parameter Type
    .Parameter Name
       
    .Parameter Force
    #>
    [CmdletBinding(DefaultParameterSetName="Type")]
    param( 
       [Parameter(ParameterSetName="Type", Mandatory=$true, ValueFromPipeline=$True, ValueFromPipelineByPropertyName=$true, Position=0)]
       [Type]$Type,
       [Parameter(Mandatory=$false, Position=1)]
       [SupportsWildCards()]
       [PSDefaultValue(Help='*')]
       [String[]]$Name ="*",
       [Switch]$Force,
       [PSDefaultValue(Help='Instance,Static,Public')]
       [System.Reflection.BindingFlags]$BindingFlags = $(if($Force){"Instance,Static,Public,NonPublic"} else {"Instance,Static,Public"}),
 
       [Parameter(Mandatory=$false)]
       [SupportsWildCards()]
       [String[]]$Attribute
 
    )
    process {
       Write-Verbose "[$($type.FullName)].GetMethods(`"$BindingFlags`")"
       Write-Verbose "[$($type.FullName)].GetConstructors(`"$BindingFlags`")"
       Write-Verbose "Filter by Name -like '$Name'"
 
       
       $Type.GetMethods($BindingFlags) + $type.GetConstructors($BindingFlags) | Where-Object {
          ($Force -or !$_.IsSpecialName -or $_.Name -notmatch "^get_|^set_") -AND 
          ($Name -eq "*" -or ($( foreach($n in $Name) { $_.Name -like $n } ) -contains $True)) -AND
          (!$Attribute -or $( foreach($n in $Attribute) { $_.CustomAttributes | ForEach { $_.AttributeType.Name -like $n -or $_.AttributeType.FullName -like $n } } ) -contains $True ) 
       }
    }
 }
 
 
 
 
 if(!($RMI = Get-TypeData System.Reflection.RuntimeMethodInfo) -or !$RMI.Members.ContainsKey("TypeName")) {
    Update-TypeData -TypeName System.Reflection.RuntimeMethodInfo -MemberName "TypeName" -MemberType ScriptProperty -Value { $this.ReflectedType.FullName }
    Update-TypeData -TypeName System.Reflection.RuntimeMethodInfo -MemberName "Definition" -MemberType ScriptProperty -Value { Get-MemberSignature $this -Simple }
    Update-TypeData -TypeName System.Reflection.RuntimeMethodInfo -MemberName "Syntax" -MemberType AliasProperty -Value "Definition"
    Update-TypeData -TypeName System.Reflection.RuntimeMethodInfo -MemberName "SafeSyntax" -MemberType ScriptProperty -Value { Get-MemberSignature $this }
 }
 
 function Get-MemberSignature {
    <#
       .Synopsis
          Get the powershell signature for calling a member.
    #>
    [CmdletBinding(DefaultParameterSetName="CallSignature")]
    param(
       [Parameter(ValueFromPipeline=$true,Mandatory=$true,Position=0)]
       [System.Reflection.MethodBase]$MethodBase,
 
       [Parameter(Mandatory=$false, Position=1)]
       [Type[]]$GenericArguments,
       
       [Parameter(ParameterSetName="CallSignature")]
       [Switch]$Simple,
       
       [Parameter(ParameterSetName="ParamBlock")]
       [Switch]$ParamBlock
    )
    process {
       if($PSCmdlet.ParameterSetName -eq "ParamBlock") { $Simple = $true }
 
       $parameters = $(
          foreach($param in $MethodBase.GetParameters()) {
             $paramType = $param.ParameterType
 
             Write-Verbose "$(if($paramType.IsGenericType){'Generic: '})$($GenericArguments)"
             if($paramType.IsGenericType -and $GenericArguments) {
                try {
                   $paramType = $paramType.GetGenericTypeDefinition().MakeGenericType( $GenericArguments )
                } catch { continue }
             }
          
             if($paramType.Name.EndsWith('&')) { $ref = '[ref]' } else { $ref = '' }
             if($paramType.IsArray) { $array = ',' } else { $array = '' }
             if($ParamBlock) { 
                '[Parameter(Mandatory=$true)]{0}[{1}]${2}' -f $ref, $paramType.ToString().TrimEnd('&'), $param.Name
             } elseif($Simple) { 
                '[{0}] {2}' -f $paramType.ToString().TrimEnd('&'), $param.Name
             } else {
                '{0}({1}[{2}]${3})' -f $ref, $array, $paramType.ToString().TrimEnd('&'), $param.Name
             }
          }
       )
       if($PSCmdlet.ParameterSetName -eq "ParamBlock") {
          $parameters -join ', '
       } elseif($MethodBase.IsConstructor) {
          "New-Object $($MethodBase.ReflectedType.FullName) $($parameters -join ', ')"
       } elseif($Simple) {
          "$($MethodBase.ReturnType.FullName) $($MethodBase.Name)($($parameters -join ', '))"
       } elseif($MethodBase.IsStatic) {
          "[$($MethodBase.ReturnType.FullName)] [$($MethodBase.ReflectedType.FullName)]::$($MethodBase.Name)($($parameters -join ', '))"
       } else {
          "[$($MethodBase.ReturnType.FullName)] `$$($MethodBase.ReflectedType.Name)Object.$($MethodBase.Name)($($parameters -join ', '))"
       }
    }
 }
 
 function Read-Choice {
    [CmdletBinding()]
    param(
       [Parameter(Mandatory=$true, ValueFromRemainingArguments=$true)]
       [hashtable[]]$Choices,
 
       [Parameter(Mandatory=$False)]
       [string]$Caption = "Please choose!",
 
       [Parameter(Mandatory=$False)]
       [string]$Message = "Choose one of the following options:",
 
       [Parameter(Mandatory=$False)]
       [int[]]$Default  = 0,
 
       [Switch]$MultipleChoice,
 
       [Switch]$Passthru
    )
    begin {
       [System.Collections.DictionaryEntry[]]$choices = $choices | % { $_.GetEnumerator() }
    }
    process {
       $Descriptions = [System.Management.Automation.Host.ChoiceDescription[]]( $(
                         foreach($choice in $choices) {
                            New-Object System.Management.Automation.Host.ChoiceDescription $choice.Key,$choice.Value
                         } 
                       ) )
 
       if(!$MultipleChoice) { [int]$Default = $Default[0] }
 
       [int[]]$Answer = $Host.UI.PromptForChoice($Caption,$Message,$Descriptions,$Default)
 
       if($Passthru) {
          Write-Verbose "$Answer"
          Write-Output  $Descriptions[$Answer]
       } else {
          Write-Output $Answer
       }
    }
 }
 
 function Read-Choice {
    <#
       .Synopsis
         Prompt the user for a choice, and return the (0-based) index of the selected item
       .Parameter Message
         This is the prompt that will be presented to the user. Basically, the question you're asking.
       .Parameter Choices
         An array of strings representing the choices (or menu items), with optional ampersands (&) in them to mark (unique) characters which can be used to select each item.
       .Parameter ChoicesWithHelp
         A Hashtable where the keys represent the choices (or menu items), with optional ampersands (&) in them to mark (unique) characters which can be used to select each item, and the values represent help text to be displayed to the user when they ask for help making their decision.
       .Parameter Default
         The (0-based) index of the menu item to select by default (defaults to zero).
       .Parameter MultipleChoice
         Prompt the user to select more than one option. This changes the prompt display for the default PowerShell.exe host to show the options in a column and allows them to choose multiple times.
         Note: when you specify MultipleChoice you may also specify multiple options as the default!
       .Parameter Caption
         An additional caption that can be displayed (usually above the Message) as part of the prompt
       .Parameter Passthru
         Causes the Choices objects to be output instead of just the indexes
       .Example
         Read-Choice "WEBPAGE BUILDER MENU"  "&Create Webpage","&View HTML code","&Publish Webpage","&Remove Webpage","E&xit"
       .Example
         [bool](Read-Choice "Do you really want to do this?" "&No","&Yes" -Default 1)
         
         This example takes advantage of the 0-based index to convert No (0) to False, and Yes (1) to True. It also specifies YES as the default, since that's the norm in PowerShell.
       .Example
         Read-Choice "Do you really want to delete them all?" @{"&No"="Do not delete all files. You will be prompted to delete each file individually."; "&Yes"="Confirm that you want to delete all of the files"}
         
         Note that with hashtables, order is not guaranteed, so "Yes" will probably be the first item in the prompt, and thus will output as index 0.  Because of thise, when a hashtable is passed in, we default to Passthru output.
    #>
    [CmdletBinding(DefaultParameterSetName="HashtableWithHelp")]
    param(
       [Parameter(Mandatory=$true, Position = 10, ParameterSetName="HashtableWithHelp")]
       [Hashtable]$ChoicesWithHelp
    ,   
       [Parameter(Mandatory=$true, Position = 10, ParameterSetName="StringArray")]
       [String[]]$Choices
    ,
       [Parameter(Mandatory=$False)]
       [string]$Caption = "Please choose!"
    ,  
       [Parameter(Mandatory=$False, Position=0)]
       [string]$Message = "Choose one of the following options:"
    ,  
       [Parameter(Mandatory=$False)]
       [int[]]$Default  = 0
    ,  
       [Switch]$MultipleChoice
    ,
       [Switch]$Passthru
    )
    begin {
       if($ChoicesWithHelp) { 
          [System.Collections.DictionaryEntry[]]$choices = $ChoicesWithHelp.GetEnumerator() | %{$_}
       }
    }
    process {
       $Descriptions = [System.Management.Automation.Host.ChoiceDescription[]]( $(
                         if($choices -is [String[]]) {
                            foreach($choice in $choices) {
                               New-Object System.Management.Automation.Host.ChoiceDescription $choice
                            } 
                         } else {
                            foreach($choice in $choices) {
                               New-Object System.Management.Automation.Host.ChoiceDescription $choice.Key, $choice.Value
                            } 
                         }
                       ) )
                       
       if(!$MultipleChoice) { [int]$Default = $Default[0] }
 
       [int[]]$Answer = $Host.UI.PromptForChoice($Caption,$Message,$Descriptions,$Default)
 
       if($Passthru -or !($choices -is [String[]])) {
          Write-Verbose "$Answer"
          Write-Output  $Descriptions[$Answer]
       } else {
          Write-Output $Answer
       }
    }
 
 }
 
 function Get-Argument {
    param(
       [Type]$Target,
         [ref]$Method,
         [Array]$Arguments
    )
    end {
       trap {
          write-error $_
          break
       }
 
       $flags = [System.Reflection.BindingFlags]"public,ignorecase,invokemethod,instance"
 
       [Type[]]$Types = @(
          foreach($arg in $Arguments) {
             if($arg -is [type]) { 
                $arg 
             }
             else {
                $arg.GetType()
             }
          } 
       )
       try {
          Write-Verbose "[$($Target.FullName)].GetMethod('$($Method.Value)', [$($Flags.GetType())]'$flags', `$null, ([Type[]]($(@($Types|%{$_.Name}) -join ','))), `$null)"
          $MethodBase = $Target.GetMethod($($Method.Value), $flags, $null, $types, $null)
          $Arguments
          if($MethodBase) {
             $Method.Value = $MethodBase.Name
          }
       } catch { }
       
       if(!$MethodBase) {
          Write-Verbose "Try again to get $($Method.Value) Method on $($Target.FullName):"
          $MethodBase = Get-Method $target $($Method.Value)
          if(@($MethodBase).Count -gt 1) {
             $i = 0
             $i = Read-Choice -Choices $(foreach($mb in $MethodBase) { @{ "$($mb.SafeSyntax) &$($i = $i+1;$i)`b`n" =  $mb.SafeSyntax } }) -Default ($MethodBase.Count-1) -Caption "Choose a Method." -Message "Please choose which method overload to invoke:"
             [System.Reflection.MethodBase]$MethodBase = $MethodBase[$i]
          }
          
          
          ForEach($parameter in $MethodBase.GetParameters()) {
             $found = $false
             For($a =0;$a -lt $Arguments.Count;$a++) {
                if($argument[$a] -as $parameter.ParameterType) {
                   Write-Output $argument[$a]
                   if($a -gt 0 -and $a -lt $Arguments.Count) {
                      $Arguments = $Arguments | Select -First ($a-1) -Last ($Arguments.Count -$a)
                   } elseif($a -eq 0) {
                      $Arguments = $Arguments | Select -Last ($Arguments.Count - 1)
                      $Arguments = $Arguments | Select -First ($Arguments.Count - 1)
                   }
                   $found = $true
                   break
                }
             }
             if(!$Found) {
                $userInput = Read-Host "Please enter a [$($parameter.ParameterType.FullName)] value for $($parameter.Name)"
                if($userInput -match '^{.*}$' -and !($userInput -as $parameter.ParameterType)) {
                   Write-Output ((Invoke-Expression $userInput) -as $parameter.ParameterType)
                } else {
                   Write-Output ($userInput -as $parameter.ParameterType)
                }
             }
          }
       }
    }
 }
 
 function Invoke-Member {
    [CmdletBinding()]
    param(        
       [parameter(position=10, valuefrompipeline=$true, mandatory=$true)]
       [allowemptystring()]
       $InputObject,
 
       [parameter(position=0, mandatory=$true)]
       [validatenotnullorempty()]
       $Member,
 
       [parameter(position=1, valuefromremainingarguments=$true)]
       [allowemptycollection()]
       $Arguments,
 
       [parameter()]
       [switch]$Static
    )
    process {
 
             if ($InputObject -is [type]) {
                 $target = $InputObject
             } else {
                 $target = $InputObject.GetType()
             }
          
             if(Get-Member $Member -InputObject $InputObject -Type Properties) {
                $_.$Member
             } 
             elseif($Member -match "ctor|constructor") {
                $Member = ".ctor"
                [System.Reflection.BindingFlags]$flags = "CreateInstance"
                $InputObject = $Null
             } 
             else {
                [System.Reflection.BindingFlags]$flags = "IgnoreCase,Public,InvokeMethod"
                if($Static) { $flags = "$Flags,Static" } else { $flags = "$Flags,Instance" }
             }
             [ref]$Member = $Member
             [Object[]]$Parameters = Get-Argument $Target $Member $Arguments
             [string]$Member = $Member.Value
 
             Write-Verbose $(($Parameters | %{ '[' + $_.GetType().FullName + ']' + $_ }) -Join ", ")
 
             try {
                Write-Verbose "Invoking $Member on [$target]$InputObject with [$($Flags.GetType())]'$flags' and [$($Parameters.GetType())]($($Parameters -join ','))"
                Write-Verbose "[$($target.FullName)].InvokeMember('$Member', [System.Reflection.BindingFlags]'$flags', `$null, '$InputObject', ([object[]]($(($Parameters | %{ '[' + $_.GetType().FullName + ']''' + $_ + ''''}) -join', '))))"
                $target.InvokeMember($Member, [System.Reflection.BindingFlags]"$flags", $null, $InputObject, $Parameters)
             } catch {
                Write-Warning $_.Exception
                 if ($_.Exception.Innerexception -is [MissingMethodException]) {
                     write-warning "Method argument count (or type) mismatch."
                 }
             }
    }
 }
 
 function Invoke-Generic {
    #.Synopsis
    [CmdletBinding()]
    param( 
       [Parameter(Position=0,Mandatory=$true,ValueFromPipelineByPropertyName=$true)]
       [Alias('On')]
       $InputObject,
 
       [Parameter(Position=1,ValueFromPipelineByPropertyName=$true)]
       [Alias('Named')]
       [string]$MethodName,
 
       [Parameter(Position=2)]
       [Alias("Types")]
       [Type[]]$ParameterTypes,
 
       [Parameter(Position=4, ValueFromRemainingArguments=$true, ValueFromPipelineByPropertyName=$true)]
       [Object[]]$WithArgs,
 
       [Switch]$Static
    )
    begin {
       if($Static) {
          $BindingFlags = [System.Reflection.BindingFlags]"IgnoreCase,Public,Static"
       } else {
          $BindingFlags = [System.Reflection.BindingFlags]"IgnoreCase,Public,Instance"
       }
    }
    process {
       $Type = $InputObject -as [Type]
       if(!$Type) { $Type = $InputObject.GetType() }
       
       if($WithArgs -and -not $ParameterTypes) {
          $ParameterTypes = $withArgs | % { $_.GetType() }
       } elseif(!$ParameterTypes) {
          $ParameterTypes = [Type]::EmptyTypes
       }   
       
       
       trap { continue }
       $MemberInfo = $Type.GetMethod($MethodName, $BindingFlags)
       if(!$MemberInfo) {
          $MemberInfo = $Type.GetMethod($MethodName, $BindingFlags, $null, $NonGenericArgumentTypes, $null)
       }
       if(!$MemberInfo) {
          $MemberInfo = $Type.GetMethods($BindingFlags) | Where-Object {
             $MI = $_
             [bool]$Accept = $MI.Name -eq $MethodName
             if($Accept){
             Write-Verbose "$Accept = $($MI.Name) -eq $($MethodName)"
                [Array]$GenericTypes = @($MI.GetGenericArguments() | Select -Expand Name)
                [Array]$Parameters = @($MI.GetParameters() | Add-Member ScriptProperty -Name IsGeneric -Value { 
                                           $GenericTypes -Contains $this.ParameterType 
                                        } -Passthru)
 
                                        $Accept = $ParameterTypes.Count -eq $Parameters.Count
                Write-Verbose "  $Accept = $($Parameters.Count) Arguments"
                if($Accept) {
                   for($i=0;$i -lt $Parameters.Count;$i++) {
                      $Accept = $Accept -and ( $Parameters[$i].IsGeneric -or ($ParameterTypes[$i] -eq $Parameters[$i].ParameterType))
                      Write-Verbose "   $Accept =$(if($Parameters[$i].IsGeneric){' GENERIC or'}) $($ParameterTypes[$i]) -eq $($Parameters[$i].ParameterType)"
                   }
                }
             }
             return $Accept
          } | Sort { @($_.GetGenericArguments()).Count } | Select -First 1
       }
       Write-Verbose "Time to make generic methods."
       Write-Verbose $MemberInfo
       [Type[]]$GenericParameters = @()
       [Array]$ConcreteTypes = @($MemberInfo.GetParameters() | Select -Expand ParameterType)
       for($i=0;$i -lt $ParameterTypes.Count;$i++){
          Write-Verbose "$($ParameterTypes[$i]) ? $($ConcreteTypes[$i] -eq $ParameterTypes[$i])"
          if($ConcreteTypes[$i] -ne $ParameterTypes[$i]) {
             $GenericParameters += $ParameterTypes[$i]
          }
          $ParameterTypes[$i] = Add-Member -in $ParameterTypes[$i] -Type NoteProperty -Name IsGeneric -Value $($ConcreteTypes[$i] -ne $ParameterTypes[$i]) -Passthru
       }
 
        $ParameterTypes | Where-Object { $_.IsGeneric }
       Write-Verbose "$($GenericParameters -join ', ') generic parameters"
 
       $MemberInfo = $MemberInfo.MakeGenericMethod( $GenericParameters )
       Write-Verbose $MemberInfo
 
       if($WithArgs) {
          [Object[]]$Arguments = $withArgs | %{ $_.PSObject.BaseObject }
          Write-Verbose "Arguments: $(($Arguments | %{ $_.GetType().Name }) -Join ', ')"
          $MemberInfo.Invoke( $InputObject, $Arguments )
       } else {
          $MemberInfo.Invoke( $InputObject )
       }
    } 
 }
 
 $xlr8r = [psobject].assembly.gettype("System.Management.Automation.TypeAccelerators")
 
 function Import-Namespace {
    [CmdletBinding()]
    param(
       [Parameter(ValueFromPipeline=$true)]
       [string]$Namespace,
 
       [Switch]$Force
    )
    end {
      Get-Type -Namespace $Namespace -Force:$Force | Add-Accelerator
    }
 }
 
 function Add-Accelerator {
    <#
       .Synopsis
          Add a type accelerator to the current session
       .Description
          The Add-Accelerator function allows you to add a simple type accelerator (like [regex]) for a longer type (like [System.Text.RegularExpressions.Regex]).
       .Example
          Add-Accelerator list System.Collections.Generic.List``1
          $list = New-Object list[string]
          
          Creates an accelerator for the generic List[T] collection type, and then creates a list of strings.
       .Example
          Add-Accelerator "List T", "GList" System.Collections.Generic.List``1
          $list = New-Object "list t[string]"
          
          Creates two accelerators for the Generic List[T] collection type.
       .Parameter Accelerator
          The short form accelerator should be just the name you want to use (without square brackets).
       .Parameter Type
          The type you want the accelerator to accelerate (without square brackets)
       .Notes
          When specifying multiple values for a parameter, use commas to separate the values. 
          For example, "-Accelerator string, regex".
          
          PowerShell requires arguments that are "types" to NOT have the square bracket type notation, because of the way the parsing engine works.  You can either just type in the type as System.Int64, or you can put parentheses around it to help the parser out: ([System.Int64])
 
          Also see the help for Get-Accelerator and Remove-Accelerator
       .Link
          http://huddledmasses.org/powershell-2-ctp3-custom-accelerators-finally/
    #>
    [CmdletBinding()]
    param(
       [Parameter(Position=0,ValueFromPipelineByPropertyName=$true)]
       [Alias("Key","Name")]
       [string[]]$Accelerator,
 
       [Parameter(Position=1,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
       [Alias("Value","FullName")]
       [type]$Type
    )
    process {
       foreach($a in $Accelerator) { 
          if($xlr8r::AddReplace) { 
             $xlr8r::AddReplace( $a, $Type) 
          } else {
             $null = $xlr8r::Remove( $a )
             $xlr8r::Add( $a, $Type)
          }
          trap [System.Management.Automation.MethodInvocationException] {
             if($xlr8r::get.keys -contains $a) {
                if($xlr8r::get[$a] -ne $Type) {
                   Write-Error "Cannot add accelerator [$a] for [$($Type.FullName)]`n                  [$a] is already defined as [$($xlr8r::get[$a].FullName)]"
                }
                Continue;
             } 
             throw
          }
       }
    }
 }
 
 function Get-Accelerator {
    <#
       .Synopsis
          Get one or more type accelerator definitions
       .Description
          The Get-Accelerator function allows you to look up the type accelerators (like [regex]) defined on your system by their short form or by type
       .Example
          Get-Accelerator System.String
          
          Returns the KeyValue pair for the [System.String] accelerator(s)
       .Example
          Get-Accelerator ps*,wmi*
          
          Returns the KeyValue pairs for the matching accelerator definition(s)
       .Parameter Accelerator
          One or more short form accelerators to search for (Accept wildcard characters).
       .Parameter Type
          One or more types to search for.
       .Notes
          When specifying multiple values for a parameter, use commas to separate the values. 
          For example, "-Accelerator string, regex".
          
          Also see the help for Add-Accelerator and Remove-Accelerator
       .Link
          http://huddledmasses.org/powershell-2-ctp3-custom-accelerators-finally/
    #>
    [CmdletBinding(DefaultParameterSetName="ByType")]
    param(
       [Parameter(Position=0, ParameterSetName="ByAccelerator", ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
       [Alias("Key","Name")]
       [string[]]$Accelerator,
 
       [Parameter(Position=0, ParameterSetName="ByType", ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
       [Alias("Value","FullName")]
       [type[]]$Type
    )
    process {
       switch($PSCmdlet.ParameterSetName) {
          "ByAccelerator" { 
             $xlr8r::get.GetEnumerator() | % {
                foreach($a in $Accelerator) {
                   if($_.Key -like $a) { $_ }
                }
             }
             break
          }
          "ByType" { 
             if($Type -and $Type.Count) {
                $xlr8r::get.GetEnumerator() | ? { $Type -contains $_.Value }
             }
             else {
                $xlr8r::get.GetEnumerator() | %{ $_ }
             }
             break
          }
       }
    }
 }
 
 function Remove-Accelerator {
    <#
       .Synopsis
          Remove a type accelerator from the current session
       .Description
          The Remove-Accelerator function allows you to remove a simple type accelerator (like [regex]) from the current session. You can pass one or more accelerators, and even wildcards, but you should be aware that you can remove even the built-in accelerators.
 
       .Example
          Remove-Accelerator int
          Add-Accelerator int Int64
 
          Removes the "int" accelerator for Int32 and adds a new one for Int64. I can't recommend doing this, but it's pretty cool that it works:
 
          So now, "$(([int]3.4).GetType().FullName)" would return "System.Int64"
       .Example
          Get-Accelerator System.Single | Remove-Accelerator
 
          Removes both of the default accelerators for System.Single: [float] and [single]
       .Example
          Get-Accelerator System.Single | Remove-Accelerator -WhatIf
 
          Demonstrates that Remove-Accelerator supports -Confirm and -Whatif. Will Print:
             What if: Removes the alias [float] for type [System.Single]
             What if: Removes the alias [single] for type [System.Single]
       .Parameter Accelerator
          The short form accelerator that you want to remove (Accept wildcard characters).
       .Notes
          When specifying multiple values for a parameter, use commas to separate the values. 
          For example, "-Accel string, regex".
 
          Also see the help for Add-Accelerator and Get-Accelerator
       .Link
          http://huddledmasses.org/powershell-2-ctp3-custom-accelerators-finally/
    #>
    [CmdletBinding(SupportsShouldProcess=$true)]
    param(
       [Parameter(Position=0, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
       [Alias("Key","FullName")]
       [string[]]$Accelerator
    )
    process {
       foreach($a in $Accelerator) {
          foreach($key in $xlr8r::Get.Keys -like $a) { 
             if($PSCmdlet.ShouldProcess( "Removes the alias [$($Key)] for type [$($xlr8r::Get[$key].FullName)]",
                                         "Remove the alias [$($Key)] for type [$($xlr8r::Get[$key].FullName)]?",
                                         "Removing Type Accelerator" )) {
                $xlr8r::remove($key)   
             }
          }
       }
    }
 }
 
 ###############################################################################
 
 $Script:CodeGenContentProperties = 'Content','Child','Children','Frames','Items','Pages','Blocks','Inlines','GradientStops','Source','DataPoints', 'Series', 'VisualTree'
 $DependencyProperties = @{}
 if(Test-Path $PSScriptRoot\DependencyPropertyCache.xml) {
    #$DependencyProperties = [System.Windows.Markup.XamlReader]::Parse( (gc $PSScriptRoot\DependencyPropertyCache.xml) )
    $DependencyProperties = Import-CliXml  $PSScriptRoot\DependencyPropertyCache.xml 
 }
 
 function Get-ReflectionModule { $executioncontext.sessionstate.module }
 
 function Set-ObjectProperties {
 [CmdletBinding()]
 param( $Parameters, [ref]$DObject )
 
    if($DObject.Value -is [System.ComponentModel.ISupportInitialize]) { $DObject.Value.BeginInit() }
 
    if($DebugPreference -ne "SilentlyContinue") { Write-Host; Write-Host ">>>> $($Dobject.Value.GetType().FullName)" -fore Black -back White }
    foreach ($param in $Parameters) {
       if($DebugPreference -ne "SilentlyContinue") { Write-Host "Processing Param: $($param|Out-String )" }
       if($param.Key -eq "DependencyProps") {
       }
       elseif ($param.Key.StartsWith("On_")) {
          $EventName = $param.Key.SubString(3)
          if($DebugPreference -ne "SilentlyContinue") { Write-Host "Event handler $($param.Key) Type: $(@($param.Value)[0].GetType().FullName)" }
          $sb = $param.Value -as [ScriptBlock]
          if(!$sb) {
             $sb = (Get-Command $param.Value -CommandType Function,ExternalScript).ScriptBlock
          }
          $Dobject.Value."Add_$EventName".Invoke( $sb );
 
 
 
       else { 
          try {
             if($DebugPreference -ne "SilentlyContinue") {
                Write-Host "Setting $($param.Key) of $($Dobject.Value.GetType().Name) to $($param.Value.GetType().FullName): $($param.Value)" -fore Gray
             }
             if(@(foreach($sb in $param.Value) { $sb -is [ScriptBlock] }) -contains $true) {
                $Values = @()
                foreach($sb in $param.Value) {
                   $Values += & (Get-ReflectionModule) $sb
                }
             } else {
                $Values = $param.Value
             }
 
             if($DebugPreference -ne "SilentlyContinue") { Write-Host ([System.Windows.Markup.XamlWriter]::Save( $Dobject.Value )) -foreground green }
             if($DebugPreference -ne "SilentlyContinue") { Write-Host ([System.Windows.Markup.XamlWriter]::Save( @($Values)[0] )) -foreground green }
 
             Set-Property $Dobject $Param.Key $Values
 
             if($DebugPreference -ne "SilentlyContinue") { Write-Host ([System.Windows.Markup.XamlWriter]::Save( $Dobject.Value )) -foreground magenta }
 
             if($DebugPreference -ne "SilentlyContinue") {
                if( $Dobject.Value.$($param.Key) -ne $null ) {
                   Write-Host $Dobject.Value.$($param.Key).GetType().FullName -fore Green
                }
             }
          }
          catch [Exception]
          {
             Write-Host "COUGHT AN EXCEPTION" -fore Red
             Write-Host $_ -fore Red
             Write-Host $this -fore DarkRed
          }
       }
 
       while($DependencyProps) {
          $name, $value, $DependencyProps = $DependencyProps
          $name = ([string]@($name)[0]).Trim("-")
          if($name -and $value) {
             Set-DependencyProperty -Element $Dobject.Value -Property $name -Value $Value
          }
       }
    }
    if($DebugPreference -ne "SilentlyContinue") { Write-Host "<<<< $($Dobject.Value.GetType().FullName)" -fore Black -back White; Write-Host }
 
    if($DObject.Value -is [System.ComponentModel.ISupportInitialize]) { $DObject.Value.EndInit() }
 
 }
 
 function Set-Property {
 PARAM([ref]$TheObject, $Name, $Values)
    $DObject = $TheObject.Value
 
    if($DebugPreference -ne "SilentlyContinue") { Write-Host ([System.Windows.Markup.XamlWriter]::Save( $DObject )) -foreground DarkMagenta }
    if($DebugPreference -ne "SilentlyContinue") { Write-Host ([System.Windows.Markup.XamlWriter]::Save( @($Values)[0] )) -foreground DarkMagenta }
 
    $PropertyType = $DObject.GetType().GetProperty($Name).PropertyType
    if('System.Windows.FrameworkElementFactory' -as [Type] -and $PropertyType -eq [System.Windows.FrameworkElementFactory] -and $DObject -is [System.Windows.FrameworkTemplate]) {
       if($DebugPreference -ne "SilentlyContinue") { Write-Host "Loading a FrameworkElementFactory" -foreground Green}
 
       [Xml]$Template = [System.Windows.Markup.XamlWriter]::Save( $DObject )
       [Xml]$Content = [System.Windows.Markup.XamlWriter]::Save( (@($Values)[0]) )
 
       $Template.DocumentElement.PrependChild( $Template.ImportNode($Content.DocumentElement, $true) ) | Out-Null
 
       $TheObject.Value = [System.Windows.Markup.XamlReader]::Parse( $Template.get_OuterXml() )
    }
    elseif('System.Windows.Data.Binding' -as [Type] -and @($Values)[0] -is [System.Windows.Data.Binding] -and !$PropertyType.IsAssignableFrom([System.Windows.Data.BindingBase])) {
       $Binding = @($Values)[0];
       if($DebugPreference -ne "SilentlyContinue") { Write-Host "$($DObject.GetType())::$Name is $PropertyType and the value is a Binding: $Binding" -fore Cyan}
 
       if(!$Binding.Source -and !$Binding.ElementName) {
          $Binding.Source = $DObject.DataContext
       }
       if($DependencyProperties.ContainsKey($Name)) {
          if($field) { 
             if($DebugPreference -ne "SilentlyContinue") { Write-Host "$($field)" -fore Blue }
             if($DebugPreference -ne "SilentlyContinue") { Write-Host "Binding: ($field)::`"$($DependencyProperties.$Name.$field.Name)`" to $Binding" -fore Blue}
 
             $DObject.SetBinding( ([type]$field)::"$($DependencyProperties.$Name.$field.Name)", $Binding ) | Out-Null
          } else {
             throw "Couldn't figure out $( @($DependencyProperties.$Name.Keys) -join ', ' )"
          }
       } else {
          if($DebugPreference -ne "SilentlyContinue") { 
             Write-Host "But $($DObject.GetType())::${Name}Property is not a Dependency Property, so it probably can't be bound?" -fore Cyan
          }
          try {
 
             $DObject.SetBinding( ($DObject.GetType()::"${Name}Property"), $Binding ) | Out-Null
 
             if($DebugPreference -ne "SilentlyContinue") { 
                Write-Host ([System.Windows.Markup.XamlWriter]::Save( $Dobject )) -foreground yellow
             }
          } catch {
             Write-Host "Nope, was not able to set it." -fore Red
             Write-Host $_ -fore Red
             Write-Host $this -fore DarkRed
          }
       }
    }
    elseif($PropertyType -ne [Object] -and $PropertyType.IsAssignableFrom( [System.Collections.IEnumerable] ) -and ($DObject.$($Name) -eq $null)) {
       if($Values -is [System.Collections.IEnumerable]) {
          if($DebugPreference -ne "SilentlyContinue") { Write-Host "$Name is $PropertyType which is IEnumerable, and the value is too!" -fore Cyan }
          $DObject.$($Name) = $Values
       } else { 
          if($DebugPreference -ne "SilentlyContinue") { Write-Host "$Name is $PropertyType which is IEnumerable, but the value is not." -fore Cyan }
          $DObject.$($Name) = new-object "System.Collections.ObjectModel.ObservableCollection[$(@($Values)[0].GetType().FullName)]"
          $DObject.$($Name).Add($Values)
       }
    }
    elseif($DObject.$($Name) -is [System.Collections.IList]) {
       foreach ($value in @($Values)) {
          try {
             $null = $DObject.$($Name).Add($value)
          }
          catch
          {
             if($_.Exception.Message -match "Invalid cast from 'System.String' to 'System.Windows.UIElement'.") {
                $null = $DObject.$($Name).Add( (New-System.Windows.Controls.TextBlock $value) )
             } else {
                Write-Error $_.Exception
             throw
             }
          }
       }
    }
    else {
       if($Values -is [System.Collections.IList] -and $Values.Count -eq 1) {
          if($DebugPreference -ne "SilentlyContinue") { Write-Host "Value is an IList ($($Values.GetType().FullName))" -fore Cyan}
          if($DebugPreference -ne "SilentlyContinue") { Write-Host "But we'll just use the first ($($Values[0].GetType().FullName))" -fore Cyan}
 
          if($DebugPreference -ne "SilentlyContinue") { Write-Host ([System.Windows.Markup.XamlWriter]::Save( $Values[0] )) -foreground White}
          try {
             $DObject.$($Name) = $Values[0]
          }
          catch [Exception]
          {
             if($_.Exception.Message -match "Invalid cast from 'System.String' to 'System.Windows.UIElement'.") {
                $null = $DObject.$($Name).Add( (TextBlock $Values[0]) )
             }else { 
                throw
             }
          }
       }
       {
          if($DebugPreference -ne "SilentlyContinue") { Write-Host "Value is just $Values" -fore Cyan}
          try {
             $DObject.$($Name) = $Values
          } catch [Exception]
          {
             if($_.Exception.Message -match "Invalid cast from 'System.String' to 'System.Windows.UIElement'.") {
                $null = $DObject.$($Name).Add( (TextBlock $values) )
             }else { 
                throw
             }
          }
       }
    }
 }
 
 function Set-DependencyProperty {
 [CmdletBinding()]
 PARAM(
    [Parameter(Position=0,Mandatory=$true)]
    $Property,
 
    [Parameter(Mandatory=$true,ValueFromPipeline=$true)]
    $Element,
 
    [Parameter()]
    [Switch]$Passthru
 )
 
 DYNAMICPARAM {
    $paramDictionary = new-object System.Management.Automation.RuntimeDefinedParameterDictionary
    $Param1 = new-object System.Management.Automation.RuntimeDefinedParameter
    $Param1.Name = "Value"
    $Param1.Attributes.Add( (New-Object System.Management.Automation.ParameterAttribute -Property @{ Position = 1 }) )
 
    if( $Property ) {
       if($Property.GetType() -eq ([System.Windows.DependencyProperty]) -or
          $Property.GetType().IsSubclassOf(([System.Windows.DependencyProperty]))) 
       {
          $Param1.ParameterType = $Property.PropertyType
       } 
       elseif($Property -is [string] -and $Property.Contains(".")) {
          $Class,$Property = $Property.Split(".")
          if($DependencyProperties.ContainsKey($Property)){
             $type = $DependencyProperties.$Property.Keys -like "*$Class"
             if($type) { 
                $Param1.ParameterType = [type]@($DependencyProperties.$Property.$type)[0].PropertyType
             }
          }
 
       } elseif($DependencyProperties.ContainsKey($Property)){
          if($Element) {
             if($DependencyProperties.$Property.ContainsKey( $element.GetType().FullName )) { 
                $Param1.ParameterType = [type]$DependencyProperties.$Property.($element.GetType().FullName).PropertyType
             }
          } else {
             $Param1.ParameterType = [type]$DependencyProperties.$Property.Values[0].PropertyType
          }
       }
       else 
       {
          $Param1.ParameterType = [PSObject]
       }
    }
    else 
    {
       $Param1.ParameterType = [PSObject]
    }
    $paramDictionary.Add("Value", $Param1)
    return $paramDictionary
 }
 PROCESS {
    trap { 
       Write-Host "ERROR Setting Dependency Property" -Fore Red
       Write-Host "Trying to set $Property to $($Param1.Value)" -Fore Red
       continue
    }
    if($Property.GetType() -eq ([System.Windows.DependencyProperty]) -or
       $Property.GetType().IsSubclassOf(([System.Windows.DependencyProperty]))
    ){
       trap { 
          Write-Host "ERROR Setting Dependency Property" -Fore Red
          Write-Host "Trying to set $($Property.FullName) to $($Param1.Value)" -Fore Red
          continue
       }
       $Element.SetValue($Property, ($Param1.Value -as $Property.PropertyType))
    } 
    else {
       if("$Property".Contains(".")) {
          $Class,$Property = "$Property".Split(".")
       }
 
       if( $DependencyProperties.ContainsKey("$Property" ) ) {
          $fields = @( $DependencyProperties.$Property.Keys -like "*$Class" | ? { $Param1.Value -as ([type]$DependencyProperties.$Property.$_.PropertyType) } )
          if($fields.Count -eq 0 ) { 
             $fields = @($DependencyProperties.$Property.Keys -like "*$Class" )
          }        
          if($fields.Count) {
             $success = $false
             foreach($field in $fields) {
                trap { 
                   Write-Host "ERROR Setting Dependency Property" -Fore Red
                   Write-Host "Trying to set $($field)::$($DependencyProperties.$Property.$field.Name) to $($Param1.Value) -as $($DependencyProperties.$Property.$field.PropertyType)" -Fore Red
                   continue
                }
                $Element.SetValue( ([type]$field)::"$($DependencyProperties.$Property.$field.Name)", ($Param1.Value -as ([type]$DependencyProperties.$Property.$field.PropertyType)))
                if($?) { $success = $true; break }
             }
             
             if(!$success) { 
                throw "food" 
             }           
          } else {
             Write-Host "Couldn't find the right property: $Class.$Property on $( $Element.GetType().Name ) of type $( $Param1.Value.GetType().FullName )" -Fore Red
          }
       }
       else {
          Write-Host "Unknown Dependency Property Key: $Property on $($Element.GetType().Name)" -Fore Red
       }
    }
    
    if( $Passthru ) { $Element }
 }
 }
 
 function Add-Struct {
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
        [string]$Name,
 
        [Parameter(Position=1, Mandatory=$true, ValueFromPipelineByPropertyName=$true, ParameterSetName = "Single")]
        [ScriptBlock]$Property,
`

