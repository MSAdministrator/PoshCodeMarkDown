---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.3
Encoding: ascii
License: cc0
PoshCode ID: 990
Published Date: 2009-04-01t22
Archived Date: 2015-11-24t02
---

# uiautomation - 

## Description

this is a further preview of what you can do with system.windows.uiautomation — the new wasp module is coming…

## Comments



## Usage



## TODO



## module

`new-uiaelement`

## Code

`#
 #
 
 
 
 
 
 [Reflection.Assembly]::Load("UIAutomationClient, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35")
 [Reflection.Assembly]::Load("UIAutomationTypes, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35")
 
 $SWA = "System.Windows.Automation"
 
 Import-Module Accelerators
 Add-Accelerator AutoElement        "$SWA.AutomationElement"            -EA SilentlyContinue
 
 $UIAFunctions = @{}
 
 $Patterns = [Reflection.Assembly]::GetAssembly([System.Windows.Automation.BasePattern]).GetTypes() | 
 Where {$_.BaseType -like "System.Windows.Automation.BasePattern"} 
 
 
 ForEach($pattern in $Patterns){
    $FullName = $pattern.FullName
    
    $pattern.GetMethods() | 
    Where { $_.DeclaringType -eq $_.ReflectedType -and !$_.IsSpecialName } | 
    ForEach {
       $Name = $($_.Name -replace 'Pattern`$')
       $Func1 = "Function:Invoke-$($Name)"
       $Func = "Function:Global:Invoke-$($Name)"
       if (test-path $Func1) { remove-item $Func1 }
       New-Item $Func -value "
       Param( $( [string]::Join( ", ", @($_.GetParameters() | % { "[$($_.ParameterType.FullName)]`$$($_.Name)" })) ) )
       Process { 
          trap { Write-Warning `"`$(`$_)`"; continue }
          `$pattern = `$_.GetCurrentPattern([$FullName]::Pattern)
          if(`$pattern) {
             `$Pattern.GetType().InvokeMember('$($_.name)','Public,Instance,InvokeMethod', `$null, `$pattern, `$PSBoundParameters.Values) 
          }
       }"
    }
    
    $pattern.GetProperties() | 
    Where { $_.DeclaringType -eq $_.ReflectedType -and $_.Name -notmatch "Cached|Current"} |
    ForEach {
       $Name = $($_.Name -replace 'Pattern`$')
       $Func1 = "Function:Get-$($Name)"
       $Func = "Function:Global:Get-$($Name)"
       if (test-path $Func1) { remove-item $Func1 }
       New-Item $Func -value "Process { 
          trap { Write-Warning `"$FullName `$_`"; continue }
          `$pattern = `$_.GetCurrentPattern([$FullName]::Pattern)
          if(`$pattern) {
             `$pattern.'$($_.name)'
          }
       }"
    }
 }
 
 Add-Accelerator Condition          "$SWA.Condition"                    -EA SilentlyContinue
 Add-Accelerator AndCondition       "$SWA.AndCondition"                 -EA SilentlyContinue
 Add-Accelerator OrCondition        "$SWA.OrCondition"                  -EA SilentlyContinue
 Add-Accelerator NotCondition       "$SWA.NotCondition"                 -EA SilentlyContinue
 Add-Accelerator PropertyCondition  "$SWA.PropertyCondition"            -EA SilentlyContinue
 
 Add-Accelerator AutoElementIds     "$SWA.AutomationElementIdentifiers" -EA SilentlyContinue
 Add-Accelerator TransformIds       "$SWA.TransformPatternIdentifiers"  -EA SilentlyContinue
 
 
 $FalseCondition = [Condition]::FalseCondition
 $TrueCondition  = [Condition]::TrueCondition
 
 Add-Type -AssemblyName System.Windows.Forms
 Add-Accelerator SendKeys           System.Windows.Forms.SendKeys       -EA SilentlyContinue
 
 function New-UIAElement {
 [CmdletBinding()]
 PARAM(
    [Parameter(ValueFromPipeline=$true)]
    [AutoElement]$Element
 ) 
 PROCESS {
    $Element | 
         Add-Member -Name Text            -Type ScriptProperty -PassThru -Value {
                   $this.GetCurrentPropertyValue([AutoElementIds]::NameProperty) 
     } | Add-Member -Name ClassName       -Type ScriptProperty -Passthru -Value { 
                   $this.GetCurrentPropertyValue([AutoElementIds]::ClassNameProperty) 
     } | Add-Member -Name FrameworkId     -Type ScriptProperty -Passthru -Value { 
                   $this.GetCurrentPropertyValue([AutoElementIds]::FrameworkIdProperty) 
     } | Add-Member -Name ProcessId       -Type ScriptProperty -Passthru -Value { 
                   $this.GetCurrentPropertyValue([AutoElementIds]::ProcessIdProperty) 
     } | Add-Member -Name ControlType     -Type ScriptProperty -Passthru -Value { 
                   $this.GetCurrentPropertyValue([AutoElementIds]::LocalizedControlTypeProperty) 
     }
     
 }     
 }
 
 function Select-Window {
 [CmdletBinding()]
 PARAM(
    [Parameter()]
    [Alias("Name")]
    [string]$Text="*"
 ,
    [Parameter()]
    [string]$ClassName="*"
 ,
    [Parameter()]
    [string]$ControlType="*"
 ,
 	[Parameter()]
    [switch]$Recurse
 ,
    [Parameter(ValueFromPipeline=$true)]
    [AutoElement]$Parent = [AutoElement]::RootElement
 ) 
    PROCESS {
       if($Recurse) {
          $Parent.FindAll( "Descendants", $TrueCondition ) | New-UIAElement |
          Where-Object { 
             ($_.ClassName   -like $ClassName) -AND
             ($_.Text        -like $Text) -AND
             ($_.ControlType -like $ControlType)
          }
       } else {
          $Parent.FindAll( "Children", $TrueCondition ) | New-UIAElement |
          Where-Object { 
             ($_.ClassName   -like $ClassName) -AND
             ($_.Text        -like $Text) -AND
             ($_.ControlType -like $ControlType)
          }
       }
    }
 }
 
 function formatter  { END {
    $input | Format-Table @{l="Text";e={$_.Text.SubString(0,25)}}, ClassName, FrameworkId -Auto
 }}
 
 
 function Invoke-Element {
 [CmdletBinding()]
 PARAM(
    [Parameter(ValueFromPipeline=$true)]
    [AutoElement]$Target
 )
    PROCESS {
       $Target.GetCurrentPattern([InvokePattern]::Pattern).Invoke()
    }
 }
 
 function Set-ElementText {
 [CmdletBinding()]
 PARAM(
    [Parameter()]
    [string]$Text
 ,
    [Parameter(ValueFromPipeline=$true)]
    [AutoElement]$Target
 )
    PROCESS {
       $Target.SetFocus();
       $textPattern = $valuePattern = $null
       try {
          $textPattern = $Target.GetCurrentPattern([TextPattern]::Pattern)
          Write-Host "textPattern:`n$($textPattern | gm)"
       } catch { }
       try {
          $valuePattern = $Target.GetCurrentPattern([ValuePattern]::Pattern)
          Write-Host "valuePattern:`n$($valuePattern | gm)"
       } catch { }
       
       $Target.SetFocus();
       
       
       if($valuePattern) {
          $valuePattern.SetValue( $Text )
       }
       if($textPattern) {
          [SendKeys]::SendWait("^{HOME}");
          [SendKeys]::SendWait("^+{END}");
          [SendKeys]::SendWait("{DEL}");
          [SendKeys]::SendWait( $Text )
       }
    }
 }
`

