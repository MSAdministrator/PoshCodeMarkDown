---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5128
Published Date: 2014-04-30t04
Archived Date: 2017-05-22t03
---

# read-choice - 

## Description

a little wrapper for promptforchoice – this version is a major change to be much pickier.

## Comments



## Usage



## TODO



## function

`read-choice`

## Code

`#
 #
 
 function Read-Choice {
    <#
       .Synopsis
          Prompt the user for a choice, and return the (0-based) index of the selected item
       .Example
          Read-Choice -Prompt "WEBPAGE BUILDER MENU"  "&Create Webpage","&View HTML code","&Publish Webpage","&Remove Webpage","E&xit"
       .Example
          [bool](Read-Choice "Do you really want to do this?" "&No","&Yes" -Default 1)
         
          This example takes advantage of the 0-based index to convert No (0) to False, and Yes (1) to True. It also specifies YES as the default, since that's the norm in PowerShell.
       .Example
          Read-Choice "Do you really want to delete them all?" @{label="&Yes"; Help="Confirm that you want to delete all of the files"},@{Label="&No"; Help="Do not delete all files. You will be prompted to delete each file individually."}
         
          Specifies the labels and help text explicitly using hashtables.
       .Example
          $Env:PSModulePath -Split ';' | Read-Choice -Passthru | Get-Item
 
          Pipes paths into Read-Choice to use as selections, and passes through the selected path to Get-Item
       .Example
          Get-Process | Where { $_.VM -gt 500MB } | Read-Choice -Multi -Label ProcessName -Value Id -Help { if($_.Path) { $_.Path } else { $_.ProcessName + " (" + $_.ID + ")" } }
          
          An advanced example dealing with pipeline input. In this example we're taking processes and rendering the name as the labels, and showing the path (or process name and ID) as help, and RETURNING the process Id of the selected processes
    #>
    [CmdletBinding(DefaultParameterSetName="InputObject")]
    param(
       [Parameter(Mandatory=$False, ParameterSetName="InputObject", ValueFromPipeline = $true)]
       [Object]$InputObject,
 
       [Parameter(Mandatory=$False, Position=0)]
       [string]$Prompt = "Choose one of the following options:",
 
       [Parameter(Mandatory=$true, Position=1, ParameterSetName="Choices")]
       [Array]$Choices,  
 
       [Parameter(Mandatory=$false, ParameterSetName="InputObject", ValueFromPipelineByPropertyName = $true)]
       [Alias("Name")]
       [String]$Label,
 
       [Parameter(Mandatory=$false, ParameterSetName="InputObject", ValueFromPipelineByPropertyName = $true)]
       [String]$Help,
 
       [Parameter(Mandatory=$false, ParameterSetName="InputObject", ValueFromPipelineByPropertyName = $true)]
       [String]$Value,
 
       [Parameter(Mandatory=$False)]
       [string]$Title = "Please choose!",
 
       [Parameter(Mandatory=$False)]
       [int[]]$Default  = 0,
 
       [Switch]$MultipleChoice,
 
       [Switch]$Sorted,
 
       [Switch]$Passthru
    )
    begin { 
       $ChoiceDescriptions = @() 
       $Output = @()
       if($PSCmdlet.ParameterSetName -eq "Choices") {
          $ChoiceDescriptions = $(
             foreach($choice in $Choices) {
                if($Choice -is [System.Collections.IDictionary]) {
                   foreach($Key in $Choice.Keys) {
                      if("Label" -like "${Key}*" -or "Name" -like "${Key}*") { 
                         $Name = $Choice.$Key
                      } elseif ("Help" -like "${Key}*" -or "Value" -like "${Key}*" -or "Expression" -like "${Key}*") {
                         $Value = $Choice.$Key
                      } else {
                         Write-Error "The key $Key is not valid. Expected `"Label`" and `"Help`""
                      }
                   }
                   if($Name -and $Value) {
                      New-Object System.Management.Automation.Host.ChoiceDescription $Name, $Value
                   } else {
                      Write-Error "The parameter $Choice is not valid. Expected `"Label`" and `"Help`" keys."
                   }
                } else {
                   New-Object System.Management.Automation.Host.ChoiceDescription "$Choice", "$Choice"
                }
                $Output += $Choice
             }
          )
       }
       
       $CalculatedLabel = $PSBoundParameters.ContainsKey('Label') -and !$Label
       $CalculatedHelp  = $PSBoundParameters.ContainsKey('Help') -and !$Help
       $CalculatedValue = $PSBoundParameters.ContainsKey('Value') -and !$Value
 
       if($PSBoundParameters.ContainsKey('Value')) {
          $Passthru = $True
       }
    }
    process {
       if($PSCmdlet.ParameterSetName -eq 'InputObject') {
          $Output   += if($CalculatedValue) { $Value } elseif($Value -and $InputObject.$Value) { $InputObject.$Value } elseif($Value) { $Value } else { $InputObject }
          $LabelText = if($CalculatedLabel) { $Label } elseif($Label -and $InputObject.$Label) { $InputObject.$Label } elseif($Label) { $Label } else { "$InputObject" }
          $HelpText  = if($CalculatedHelp)  { $Help  } elseif($Help -and $InputObject.$Help)   { $InputObject.$Help  } elseif($Help)  { $Help  } else { $LabelText } 
 
          if($LabelText -and $HelpText) {
             $ChoiceDescriptions += New-Object System.Management.Automation.Host.ChoiceDescription $LabelText, $HelpText
          }
       }
    }
    end {
       if(@($ChoiceDescriptions).Count -eq 0) {
          Write-Error "There were no choices generated, no input"
          return
       } elseif (@($ChoiceDescriptions).Count -eq 1) {
          return $Output
       }
 
 
       [string[]]$Labels = $ChoiceDescriptions | % { $_.Label }
       $Keys = @()
       for($l =0; $l -lt $Labels.Count; $l++) {
          if($Labels[$l].IndexOf('&') -ge 0) {
             $Keys += $Labels[$l][($Labels[$l].IndexOf('&')+1)]
          }
       }
       for($l =0; $l -lt $Labels.Count; $l++) {
          if($Labels[$l].IndexOf('&') -lt 0) {
             for($i = 0; $i -lt $Labels[$l].Length; $i++) {
                if($Keys -notcontains $Labels[$l][$i]) {
                   $Keys += $Labels[$l][$i]
                   $Labels[$l] = $Labels[$l].Insert($i,'&')
                   $ChoiceDescriptions[$l] = New-Object System.Management.Automation.Host.ChoiceDescription $Labels[$l], $ChoiceDescriptions[$l].HelpMessage
                   break
                }
             }
          }
       }
       for($l =0; $l -lt $Labels.Count; $l++) {
          if($Labels[$l].IndexOf('&') -lt 0) {
             foreach($i in 49..57+66..90) {
                if($Keys -notcontains [string][char]$i) {
                   $Keys += [string][char]$i
                   $Labels[$l] = '{0}(&{1})' -f $Labels[$l], ([string][char]$i)
                   $ChoiceDescriptions[$l] = New-Object System.Management.Automation.Host.ChoiceDescription $Labels[$l], $ChoiceDescriptions[$l].HelpMessage
                   break
                }
             }
          }
       }
       if($ChoiceDescriptions.Length -gt 34 -and $Labels -notmatch '&') {
          Write-Warning "There are too many choices, some may be unpickable!"
       }
 
       if($Sorted) {
          $Passthru = $True
          $Max = 1000
          $Indexes = $Labels | %{ if(($amp = $_.IndexOf('&')) -lt 0) { ($Max++) } else { [int][byte][char]"$($_[($amp+1)])".ToUpperInvariant() } }
          [Array]::Sort($Indexes.Clone(), $Output)
          [Array]::Sort($Indexes, $ChoiceDescriptions)
       }
 
       if(!$MultipleChoice) { [int]$Default = $Default[0] }
 
       [int[]]$Answer = $Host.UI.PromptForChoice($Title,$Prompt,$ChoiceDescriptions,$Default)
 
       if($Passthru) {
          Write-Verbose "$Answer"
          Write-Output  $Output[$Answer]
       } else {
          Write-Output $Answer
       }
    }
 }
`

