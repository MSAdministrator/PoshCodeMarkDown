---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1195
Published Date: 2011-07-06t22
Archived Date: 2011-11-20t07
---

# speech recognition 2 - 

## Description

this version of the script supports using “*” to capture dictation. the result is that you can now write macros to look up words online, or pass parameters to a function (within the constraints of your voice recognition accuracy).

## Comments



## Usage



## TODO



## module

`update-speechcommands`

## Code

`#
 #
 $null = [Reflection.Assembly]::LoadWithPartialName("System.Speech")
 
    $Global:SpeechModuleSpeaker = new-object System.Speech.Synthesis.SpeechSynthesizer
    $Global:SpeechModuleListener = new-object System.Speech.Recognition.SpeechRecognizer
 }
 
 $Script:SpeechModuleMacros = @{}
 $Script:SpeechModuleMacros.Add("Stop Listening", { $script:listen = $false; Suspend-Listening })
 $Script:SpeechModuleComputerName = ${Env:ComputerName}
 
 function Update-SpeechCommands {
 #.Synopsis
 #.Description
    $choices = new-object System.Speech.Recognition.Choices
    foreach($choice in $Script:SpeechModuleMacros.GetEnumerator()) {
    
       $g = New-Object System.Speech.Recognition.GrammarBuilder
       $phrases = @($choice.Key -split "\s*\*\s*")
       for($i=0;$i -lt $phrases.Count;$i++) {
          if($phrases[$i].Length -gt 0) {
             $g.Append( $phrases[$i] )
             if($i+1 -lt $phrases.Count) {
                $g.AppendDictation()
             }
          } elseif($i -eq 0) {
             $g.AppendDictation()
          }
       }
       $choices.Add( (New-Object System.Speech.Recognition.SemanticResultValue $g, $choice.Value.ToString()).ToGrammarBuilder() )
    }
 
    if($VerbosePreference -ne "SilentlyContinue") { $Script:SpeechModuleMacros.Keys |
       ForEach-Object { Write-Host "$Computer, $_" -Fore Cyan } }
 
    $builder = New-Object System.Speech.Recognition.GrammarBuilder "$Computer, "
    $builder.Append((New-Object System.Speech.Recognition.SemanticResultKey "Commands", $choices.ToGrammarBuilder()))
    $grammar = new-object System.Speech.Recognition.Grammar $builder
    $grammar.Name = "Power VoiceMacros"
 
    Unregister-Event "SpeechModuleCommandRecognized" -ErrorAction SilentlyContinue
    $null = Register-ObjectEvent $grammar SpeechRecognized `
             -SourceIdentifier "SpeechModuleCommandRecognized" `
             -Action { $_ = $event.SourceEventArgs.Result.Text; iex $event.SourceEventArgs.Result.Semantics.Item("Commands").Value  }
    
    $Global:SpeechModuleListener.UnloadAllGrammars()
    $Global:SpeechModuleListener.LoadGrammarAsync( $grammar )
 }
 
 function Add-SpeechCommands {
 #.Synopsis
 #.Parameter CommandText
    [CmdletBinding()]
    Param([hashtable]$VoiceMacros,[string]$Computer=$Script:SpeechModuleComputerName)
    
    $Script:SpeechModuleMacros += $VoiceMacros
    $Script:SpeechModuleComputerName = $Computer
    Update-SpeechCommands
 }
 
 function Remove-SpeechCommands {
 #.Synopsis
 #.Parameter CommandText
    Param([string[]]$CommandText)
    foreach($command in $CommandText) { $Script:SpeechModuleMacros.Remove($Command) }
    Update-SpeechCommands
 }
 
 function Clear-SpeechCommands {
 #.Synopsis
 #.Parameter CommandText
    $Script:SpeechModuleMacros = @{}
    $Script:SpeechModuleMacros.Add("Stop Listening", { Suspend-Listening })
    Update-SpeechCommands
 }
 
 
 function Start-Listening {
 #.Synopsis
    $Global:SpeechModuleListener.Enabled = $true
    Say "Speech Macros are $($Global:SpeechModuleListener.State)"
    Write-Host "Speech Macros are $($Global:SpeechModuleListener.State)"
 }
 function Suspend-Listening {
 #.Synopsis
    $Global:SpeechModuleListener.Enabled = $false
    Say "Speech Macros are disabled"
    Write-Host "Speech Macros are disabled"
 }
 
 function Out-Speech {
 #.Synopsis
 #.Description
 #.Parameter InputObject
    Param( [Parameter(ValueFromPipeline=$true)][Alias("IO")]$InputObject )
    $null = $Global:SpeechModuleSpeaker.SpeakAsync(($InputObject|Out-String))
 }
 
 function Remove-SpeechXP {
 #.Synopis
    $Global:SpeechModuleListener.Dispose(); $Global:SpeechModuleListener = $null
    $Global:SpeechModuleSpeaker.Dispose(); $Global:SpeechModuleSpeaker = $null
 }
 
 set-alias asc Add-SpeechCommands
 set-alias rsc Remove-SpeechCommands
 set-alias csc Clear-SpeechCommands
 set-alias say Out-Speech
 set-alias listen Start-Listener
 Export-ModuleMember -Function * -Alias * -Variable SpeechModuleListener, SpeechModuleSpeaker
`

