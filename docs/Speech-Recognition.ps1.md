---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2671
Published Date: 2012-05-10t23
Archived Date: 2017-03-18t12
---

# speech recognition - 

## Description

this is an update to my “speech.psm1” script module for doing voice/speech recognition. with this version, speech macros will be executed asynchronously, so it doesn’t tie up the shell for the duration

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
       $choices.Add( (New-Object System.Speech.Recognition.SemanticResultValue $g, 
                      $choice.Value.ToString()).ToGrammarBuilder() )
    }
 
    if($VerbosePreference -ne "SilentlyContinue") { $Script:SpeechModuleMacros.Keys | ForEach-Object { Write-Host "$($Script:SpeechModuleComputerName), $_" -Fore Cyan } }
 
    $builder = New-Object System.Speech.Recognition.GrammarBuilder "$($Script:SpeechModuleComputerName), "
    $builder.Append((New-Object System.Speech.Recognition.SemanticResultKey "Commands", $choices.ToGrammarBuilder()))
    $grammar = new-object System.Speech.Recognition.Grammar $builder
    $grammar.Name = "Power VoiceMacros"
 
    Unregister-Event "SpeechModuleCommandRecognized" -ErrorAction SilentlyContinue
    $null = Register-ObjectEvent $grammar SpeechRecognized `
                -SourceIdentifier "SpeechModuleCommandRecognized" `
                -Action { iex $event.SourceEventArgs.Result.Semantics.Item("Commands").Value }
    
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
 
 function Suspend-Speech {
    $Global:SpeechModuleSpeaker.Pause()
 }
 function Resume-Speech {
    $Global:SpeechModuleSpeaker.Resume()
 }
 
 function Out-Speech {
 #.Synopsis
 #.Description
 #.Parameter InputObject
    Param( 
    [Parameter(Position=0)]
    [Object[]]
    $Property
 ,
    [Parameter()]
    [String]
    $Expand
 ,
    [Parameter()]
    [Object]
    $GroupBy
 ,
    [Parameter(ValueFromPipeline=$true)]
    [Alias("IO")]
    [PSObject]$InputObject
 ,
    [Parameter()]
    [String]
    $View
 ,
    [Parameter()]
    [Switch]
    $Async
    )
    begin {
       if($Async) {
          $PSBoundParameters.Remove("Async") | out-null
       }
       $InputCollection = new-object System.Collections.Generic.List[PSObject]
    }
    process {
       $InputCollection.Add($InputObject)
    }
    end {
       $PSBoundParameters["InputObject"] = $InputCollection
       
       if(!$Async) {
          
          Format-List @PSBoundParameters | ForEach-Object { 
             if($_.GetType().Name -eq "FormatEntryData") { $_; sleep -milli 500} else {$_}
          } | Out-String -stream | ForEach-Object {
             Write-Host $_
             $Global:SpeechModuleSpeaker.Speak($_)
          }
       } else {
          $speech = (Format-List @PSBoundParameters | Out-String -width 1e4) -replace "`r`n`r`n","`r`n`r`n`r`n"
          Write-Host $speech
          $Global:SpeechModuleSpeaker.SpeakAsync($speech) | Out-Null
       }
    }
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
 set-alias listen Start-Listening
 Export-ModuleMember -Function * -Alias * -Variable SpeachModuleListener, SpeechModuleSpeaker
 
 ###################################################################################################
 ###################################################################################################
 #
`

