---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 3.1
Encoding: ascii
License: cc0
PoshCode ID: 5297
Published Date: 2015-07-10t22
Archived Date: 2015-04-24t02
---

# consoletoys,psm1 - 

## Description

show-consolemenu shows a vertical “menu” in the console and allows you to pick numeric items from it.

## Comments



## Usage



## TODO



## function

`show-consolemenu`

## Code

`#
 #
 function Show-ConsoleMenu {
    <#
       .Synopsis
         Displays a menu in the console and returns the selection
       .Description
         Displays a numbered list in the console, accepts a typed number from the user, and returns it.
       .Example
         ls | Show-ConsoleMenu -Title "Please pick a file to delete:" -Passthru | rm -whatif
       
         Description
         -----------
         Creates a menu showing a line for every file, and outputs the selected file.
       .Example
         if(Test-Path $Profile) {
            switch(Show-ConsoleMenu "Profile exists:" "Delete it!","Rename it with 01","Abort") {
               1 { rm $Profile -whatif }
               2 { mv $Profile [IO.Path]::ChangeExtension($Profile,"01.ps1") }
               3 { return }
            }
         }
       
         Description
         -----------
         Shows how to use the return value without the Passthru switch.
         This example would check if you have a profile, and if you do, would offer you the choice of removing or renaming it.
    #>
    param(
       [Parameter(ValueFromPipeline=$true,Position=2)]
       [Alias("InputObject")]
       [PSObject[]]$Choices,
       [Parameter(Position=1)]
       [Alias("Title")]
       [string]$Caption,
       [Parameter(Position=3)]
       [ScriptBlock]$Expression=({$_}),
       [Alias("Footer")]
       [string]$Prompt,
       [int]$indentLeft=4,
       [Switch]$Passthru,
       [Switch]$UseBufferBox,
       [Switch]$MultiSelect
    )
    begin {
       $allchoices = New-Object System.Collections.Generic.List[PSObject]
    }
    process {
       if($choices) {
          $allchoices.AddRange($choices)
       }
    }
    end {
       $Format = "{0:D1}"; 
       $Digits = ($allchoices.Count + 1).ToString().Length
       $Format = "{0:D${Digits}}" 
 
       $numbersWarning = "Numbers Only Please"
       $rangeWarning   = "Only {0:D${Digits}}..{1:D${Digits}} Please" -f 1, ($allchoices.Count)
 
       for($i=0; $i -lt $allchoices.Count; $i++) {
          $allchoices[$i] = Add-Member -Input $allchoices[$i] -Type NoteProperty -Name ConsoleMenuKey -Value $($format -f ($i+1)) -Passthru
       }
 
       Write-Debug "There are $($allChoices.Count) choices, so we'll use $Digits digits"
       $width = [Math]::Max("$Prompt".Length, "$Caption".Length)
       $width = [Math]::Max($width, $numbersWarning.Length)
 
       $menu = $allchoices | Format-Table -HideTableHeader @{E="ConsoleMenuKey";A="Right";W=$indentLeft}, @{E=$Expression;A="Left"} -Force |
                             Out-String -Stream | ForEach { $line = $_.TrimEnd(); if($width -lt $line.Length) { $width = $line.Length }; $line }
       $menu = $menu[0..$($menu.Length-2)]
 
       if($UseBufferBox) {
          $OriginalCursorPosition = $Host.UI.RawUI.CursorPosition
          $Width += $(if($indentLeft -ge 4) { $indentLeft } else { $Width += 4 })
          $menu = $menu | Where { $_.length }
          $Height = $Menu.Length + $(if($Prompt){5}else{4})
          Reset-Buffer -Position $OriginalCursorPosition -Title $Caption -Height $Height -Width $Width -ShowInput
          $menu | Out-Buffer 
          Set-BufferInputLine
       } else {
          Write-Host ("`n" + (" " * ($indentLeft/2)) + $Caption + "`n")  -ForegroundColor $Host.PrivateData.VerboseForegroundColor  -BackgroundColor $Host.PrivateData.VerboseBackgroundColor
          Write-Host $menu
       }
       
       do {
          if($Prompt) {
             if($UseBufferBox) {
                Reset-CursorPosition $BufferPromptPosition -LinesOffset 2
                Out-Buffer $Prompt -ForegroundColor $Host.PrivateData.VerboseForegroundColor  -BackgroundColor $Host.PrivateData.VerboseBackgroundColor -FixInput
             } else {
                Write-Host $Prompt -ForegroundColor $Host.PrivateData.VerboseForegroundColor  -BackgroundColor $Host.PrivateData.VerboseBackgroundColor
             }
          }      
          [string]$PreviousKeys=""
          do { 
             $Key = $Host.UI.RawUI.ReadKey("IncludeKeyDown,NoEcho").Character
             try { 
                [int][string]$choice = "${PreviousKeys}${Key}"
                $index = $choice - 1
                $PreviousKeys = "${PreviousKeys}${Key}"
                if($UseBufferBox) {
                   Set-BufferInputLine $PreviousKeys
                } else {
                    Write-Host $Key -NoNewline
                }
             } catch { 
                if(63 -eq [int][char]$Key) {
                   if($UseBufferBox) {
                      $menu | Out-Buffer 
                   } else {
                      Write-Host $menu
                   }
                } elseif(13,27,0 -notcontains [int][char]$Key) {
                   if($UseBufferBox) {
                      $menu | Out-Buffer 
                      Out-Buffer $numbersWarning -ForegroundColor $Host.PrivateData.WarningForegroundColor -BackgroundColor $Host.PrivateData.WarningBackgroundColor -FixInput
                   } else {
                      Write-Warning $numbersWarning
                   }
                }
                [int][string]$choice = "${PreviousKeys}"
             }
             if($choice -gt $allchoices.Count) {
                $PreviousKeys = ""
                if($UseBufferBox) {
                   $menu | Out-Buffer
                   Out-Buffer $rangeWarning -ForegroundColor $Host.PrivateData.WarningForegroundColor -BackgroundColor $Host.PrivateData.WarningBackgroundColor -FixInput
                } else {
                   Write-Warning $rangeWarning
                }
             }
          } while( $PreviousKeys.Length -lt $Digits -and (13,27 -notcontains [int][char]$Key))
 
          if($index -ge 0 -and $index -lt $allchoices.Count) {
             if($UseBufferBox) {
                Reset-CursorPosition $BufferPromptPosition -LinesOffset 2
             } else {
                Write-Host
             }
             if($Passthru) { 
                $allchoices[$index] 
             } else { 
                $choice
             }
          }
       } while($key -ne [char]13 -and $MultiSelect)
    }
 }
 
 
 
 ####################################################################################################
 ##
 ####################################################################################################
 $global:BoxChars = new-object PSObject -Property @{
    'HorizontalDouble'            = ([char]9552).ToString()
    'VerticalDouble'              = ([char]9553).ToString()
    'TopLeftDouble'               = ([char]9556).ToString()
    'TopRightDouble'              = ([char]9559).ToString()
    'BottomLeftDouble'            = ([char]9562).ToString()
    'BottomRightDouble'           = ([char]9565).ToString()
    'HorizontalDoubleSingleDown'  = ([char]9572).ToString()
    'HorizontalDoubleSingleUp'    = ([char]9575).ToString()
    'Horizontal'                  = ([char]9472).ToString()
    'Vertical'                    = ([char]9474).ToString()
    'TopLeft'                     = ([char]9484).ToString()
    'TopRight'                    = ([char]9488).ToString()
    'BottomLeft'                  = ([char]9492).ToString()
    'BottomRight'                 = ([char]9496).ToString()
    'Cross'                       = ([char]9532).ToString()
    'VerticalDoubleRightSingle'   = ([char]9567).ToString()
    'VerticalDoubleLeftSingle'    = ([char]9570).ToString()
 }
 $global:RectType = "system.management.automation.host.rectangle"
 
 
 function Reset-Buffer {
    param(
       $Position = $Host.UI.RawUI.WindowPosition,
       [int]$Height = $Host.UI.RawUI.WindowSize.Height,
       [int]$Width = $Host.UI.RawUI.WindowSize.Width,
       [int]$Padding = 1,
       $ForegroundColor = $Host.UI.RawUI.ForegroundColor,
       $BackgroundColor = $Host.UI.RawUI.BackgroundColor,
       $BorderColor     = "Yellow",
       [switch]$NoBorder,
       [switch]$ShowInput,
       [string]$Title = ""
    )
 
    $global:BufferHeight          = $Height
    $global:BufferWidth           = $Width
    $global:BufferPadding         = $Padding
    $global:BufferForegroundColor = $ForegroundColor 
    $global:BufferBackgroundColor = $BackgroundColor 
    $global:BufferBorderColor     = $BorderColor     
 
    if($NoBorder) {
       $global:BufferBoxSides = 0
    } else {
       $global:BufferBoxSides = 2
    }
    if($ShowInput) {
       $global:BufferHeight -= 2
    }
 
    Write-Debug "RESET: Position: $Position Left: $Left Top: $Top Width: $Width Height: $Height Offset: $Offset"
 
    $Host.UI.RawUI.SetBufferContents($Position,(New-BufferBox $BufferHeight $BufferWidth -Title:$Title -NoBorder:$NoBorder -ShowInput:$ShowInput -Background $BufferBackgroundColor -Border $BufferBorderColor))
   
    
    $global:BufferPosition = $Position   
    $global:BufferPosition.X += $global:BufferPadding + ($global:BufferBoxSides/2)
    $global:BufferPosition.Y += $global:BufferHeight - 2
    $global:BufferPromptPosition = $BufferPosition
    $global:BufferPromptPosition.Y += 2
 }
 
 function New-BufferBox {
    param(
       [int]$Height = $global:BufferHeight, 
       [int]$Width = $global:BufferWidth,
       $Title = "",
       [switch]$NoBorder,
       [switch]$ShowInput,
       $BackgroundColor = $global:BufferBackgroundColor,
       $BorderColor = $global:BufferBorderColor 
    )
    $Width = $Width - $global:BufferBoxSides
    
    $LineTop =( $global:BoxChars.HorizontalDouble * 2) + $Title `
             + $($global:BoxChars.HorizontalDouble * ($Width - ($Title.Length+2)))
    
    $LineField = ' ' * $Width
    $LineBottom = $global:BoxChars.HorizontalDouble * $Width
    $LineSeparator = $global:BoxChars.Horizontal * $Width
    
    if(!$NoBorder) {
       $LineField = $global:BoxChars.VerticalDouble + $LineField + $global:BoxChars.VerticalDouble
       $LinePrompt = $global:BoxChars.VerticalDouble + $LinePrompt + $global:BoxChars.VerticalDouble
       $LineBottom = $global:BoxChars.BottomLeftDouble + $LineBottom + $global:BoxChars.BottomRightDouble
       $LineTop = $global:BoxChars.TopLeftDouble + $LineTop + $global:BoxChars.TopRightDouble
       $LineSeparator = $global:BoxChars.VerticalDoubleRightSingle + $LineSeparator + $global:BoxChars.VerticalDoubleLeftSingle
    }
 
    if($ShowInput) {
       $box = &{$LineTop;1..($Height - 2) |% {$LineField};$LineSeparator;$LinePrompt;$LineBottom}
    } else {
       $box = &{$LineTop;1..($Height - 2) |% {$LineField};$LineBottom}
    }
    $boxBuffer = $Host.UI.RawUI.NewBufferCellArray($box,$BorderColor,$BackgroundColor)
    return ,$boxBuffer
 }
 
 function Move-Buffer {
    param(
       $Position = $global:BufferPosition,
       [int]$Left = $($global:BufferBoxSides/2),
       [int]$Top = (2 - $global:BufferHeight),
       [int]$Width = $global:BufferWidth - $global:BufferBoxSides,
       [int]$Height = $global:BufferHeight,
       [int]$Offset = -1
    )
    Write-Debug "MOVE: Position: $Position Left: $Left Top: $Top Width: $Width Height: $Height Offset: $Offset"
 
    $Position.X += $Left
    $Position.Y += $Top
    $Rect = New-Object $RectType $Position.X, $Position.Y, ($Position.X + $width), ($Position.Y + $height -1)
    $Position.Y += $OffSet
    $Host.UI.RawUI.ScrollBufferContents($Rect, $Position, $Rect, (new-object System.Management.Automation.Host.BufferCell(' ',$global:BufferForegroundColor,$global:BufferBackgroundColor,'complete')))
 }
 
 function Out-Buffer {
    param(
       [Parameter(ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true,Position=0,Mandatory=$true)]
       $Message, 
       [Parameter(ValueFromPipelineByPropertyName=$true)]
       $ForegroundColor = $global:BufferForegroundColor, 
       [Parameter(ValueFromPipelineByPropertyName=$true)]
       $BackgroundColor = $global:BufferBackgroundColor, 
       
       [switch]$NoScroll,
       
       [Parameter(ValueFromPipelineByPropertyName=$true)]
       $Position = $global:BufferPosition,
       [Parameter(ValueFromPipelineByPropertyName=$true)]
       [int]$Left = 0,
       [Parameter(ValueFromPipelineByPropertyName=$true)]
       [Parameter(ValueFromPipelineByPropertyName=$true)]
       [Parameter(ValueFromPipelineByPropertyName=$true)]
       [Parameter(ValueFromPipelineByPropertyName=$true)]
       [int]$Offset = $( 0 - ("$Message".Split("`n").Count)),
       [Switch]$FixInput
    )
    process {
       $lineCount = $Message.Split("`n").Count
 
       $Width = $Width - ($global:BufferPadding * 2)
       if(!$NoScroll){ Move-Buffer $Position $Left $Top $Width $Height $Offset }
       
       $MessageBuffer = New-Object "System.Management.Automation.Host.BufferCell[,]" $lineCount, $width
       
       $index = 0
       foreach( $line in $Message.Split("`n") ) {
          $Buffer = $Host.UI.RawUI.NewBufferCellArray( @($line.Trim("`r").PadRight($Width)), $ForegroundColor, $BackgroundColor )
          for($i = 0; $i -lt $width; $i++) {
             $MessageBuffer[$index,$i] = $Buffer[0,$i]
          }
          $index++
       }
       
       $Y = $global:BufferPosition.Y
       $global:BufferPosition.Y -= $lineCount - 1
       $Host.UI.RawUI.SetBufferContents($global:BufferPosition,$MessageBuffer)
       $global:BufferPosition.Y = $Y
 
       if($FixInput) {
          Set-BufferInputLine
       }
    }
 }
 
 function Set-BufferInputLine {
    param([String]$Line = "")
    
    $CursorPosition = $BufferPromptPosition
    $CursorPosition.X += $line.Length
    
    $Prompt = $Host.UI.RawUI.NewBufferCellArray( @($PromptText),$global:BufferForegroundColor, $global:BufferBackgroundColor)
    $Host.UI.RawUI.SetBufferContents( $BufferPromptPosition, $prompt )
    $Host.UI.RawUI.CursorPosition = $CursorPosition
 }
 
 function Reset-CursorPosition {
    param(
       $CursorPosition = $BufferPromptPosition,
       $LinesOffset = 2
    )
    $CursorPosition.Y += $LinesOffset
    $CursorPosition.X = 0
    $Host.UI.RawUI.CursorPosition = $CursorPosition
 }
 
 function Test-DisplayBox {
    $Position = $Host.UI.RawUI.WindowPosition
    $Position.X += 10
    $Position.Y += 3
 
    Reset-Buffer $Position 20 66 3 -ForegroundColor 'Gray' -BackgroundColor 'Black' -BorderColor 'Green'
    Out-Buffer 'Greetings!' 'Yellow' 'black'
    sleep -m 600
    Out-Buffer '' 'Gray' 'black'
    sleep -m 600
    Out-Buffer '- - - Thank you for running this simple demo script! - - -' 'Green' 'black'
    sleep -m 600
    Out-Buffer '' 'Gray' 'black'
    sleep -m 600
    Out-Buffer "We are demonstrating how the scroll buffer works: you can`nadd more than one line at a time, but you don't really`nneed to, since they add almost as fast one at a time." 'white' 'black'
    sleep -m 3000
    Out-Buffer '' 'Gray' 'black'
    Out-Buffer 'That is, as long as you don''t have any delay, you can just' 'white' 'black'
    Out-Buffer 'write out as many lines as you like, one-at-a-time, like' 'white' 'black'
    Out-Buffer 'this, and there is really no downside to doing that.' 'white' 'black'
    sleep -m 3000
    Out-Buffer '' 'Gray' 'black'
    Out-Buffer 'Right? '.PadRight(58,"-") 'Red' 'black'   
    Out-Buffer '' 'Gray' 'black'
    sleep -m 600
    Out-Buffer 'It''s clearly not as slick to just pop in multiple lines' 'white' 'black'
    sleep -m 1200
    Out-Buffer 'with absolutely no scroll delay, and it makes it a little ' 'white' 'black'
    sleep -m 1200
    Out-Buffer 'hard to tell what you have read already, but that''s ok.' 'white' 'black'
    sleep -m 1200
    Out-Buffer '' 'Gray' 'black'
    sleep -m 600
    Out-Buffer 'If you delay between paragraphs.' 'Red' 'black'   
    sleep -m 600
    Out-Buffer '' 'Gray' 'black'
    sleep -m 600
    Out-Buffer 'But: don''t scroll off-screen faster than I can read!' 'Yellow' 'black'   
    sleep -m 600
    Out-Buffer '' 'Gray' 'black'
 }
 
 ####################################################################################################
 ##
 ####################################################################################################
 function Test-BufferBox {
    param(
       $title = "PowerShell Interactive Buffer Demo"
    )
 
    Reset-Buffer -ShowInput -Title $Title
 
    ###################################################################################################
    $noise = $MyInvocation.MyCommand.Definition -split "`n"
    $index = 0; 
    $random = New-Object "Random"
    [regex]$chunker = @'
 [^ \"']+|([\"'])[^\\1]*?\\1[^ \"']*|([\"'])[^\\1]*$| $
 '@
    Out-Buffer "  " -Fore DarkGray -Back Cyan
    Out-Buffer "     This is just a demo. " -Fore Black -Back Cyan
    Out-Buffer "     We will stream in the source code of this script ... " -Fore Black -Back Cyan
    Out-Buffer "     And you can type at the bottom while it's running. " -Fore Black -Back Cyan
    Out-Buffer "     Imagine this as an n-way chat application like IRC, except that " -Fore Black -Back Cyan
    Out-Buffer "  FOR THIS PERFORMANCE ONLY: " -Fore Black -Back Cyan
    Out-Buffer "         The part of your chatting friends is played by my source code. " -Fore DarkGray -Back Cyan
    Out-Buffer "  " -Fore DarkGray -Back Cyan
    Out-Buffer "     Press ESC to exit, or enter 'exit' and hit Enter" -Fore Black -Back Cyan
    Out-Buffer "  " -Fore DarkGray -Back Cyan
    Set-BufferInputLine
    $line=""
    $exit = $false
     
    switch(Show-ConsoleMenu "What would you like to do now?" "Continue the demo","Stop the demo","Exit PowerShell" -UseBuffer) {
    }
 
    while(!$exit){
       while(!$exit -and $Host.UI.RawUI.KeyAvailable) { 
          $char = $Host.UI.RawUI.ReadKey("IncludeKeyUp,IncludeKeyDown,NoEcho"); 
          if($char.KeyDown) {
          switch([int]$char.Character) {
                   if($line.Trim() -eq "exit") { $exit = $true; break; }
    ###################################################################################################
    ###################################################################################################
                iex "Out-Buffer `"$line`" -Fore Red";                                     #############
    ###################################################################################################
    ###################################################################################################
                $line = "";
                Set-BufferInputLine
                break;
             }
                $exit = $true; break;
             }
                if($line.Length -gt 0) {
                   [Array]$words = $chunker.Matches($line)
                   if($words.Count -ge 1) {
                      Out-Buffer ($Words | Out-String) -Fore Black -Back White
                      $lastWord = $words[$($words.Count-1)].Value
                      $trim = $lastWord.TrimEnd("`r","`n")
                      $replacement = TabExpansion -Line $line -LastWord ($lastWord.Trim() -replace '"')
                      Out-Buffer ($replacement | Out-String) -Fore Black -Back White
                      $line = $line.SubString(0, $line.Length - $lastWord.Length) + @($replacement)[0]
                      Set-BufferInputLine $line
                   }
                }         
                break;
             }
                if($line.Length -gt 0) {
                   $line = $line.SubString(0,$($line.Length-1))
                }
                Set-BufferInputLine $line
                break;
             }         
             0 { 
                break;
             }
             default {
                $line += $char.Character
                Set-BufferInputLine $line
             }
          }
          }
       }
       if($random.NextDouble() -gt 0.75) {
          $noise[($index)..($index++)] | Out-Buffer
          if($index -ge $noise.Length){$index = 0}
       }
       sleep -milli 100
    }
    Reset-CursorPosition $BufferPromptPosition
 }
`

