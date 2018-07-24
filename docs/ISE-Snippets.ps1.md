---
Author: poetter
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 786
Published Date: 2009-01-04t18
Archived Date: 2017-03-25t01
---

# ise-snippets - 

## Description

ise-snippets module v 1.0

## Comments



## Usage



## TODO



## module

`add-snippet`

## Code

`#
 #
 ##############################################################################################################
 ##############################################################################################################
 ##############################################################################################################
 
 
 ##############################################################################################################
 ##############################################################################################################
 function Add-Snippet
 {
     $snippets = @{
         "function" = @( "function FUNCTION_NAME", "{", "}" )
         "param" = @( "param ``", "(", ")" )
         "if" = @( "if ( CONDITION )", "{", "}" )
         "else" = @( "else", "{", "}" )
         "elseif" = @( "elseif ( CONDITION )", "{", "}" )
         "foreach" = @( "foreach ( ITEM in COLLECTION )", "{", "}" )
         "for" = @( "foreach ( INIT; CONDITION; REPEAT )", "{", "}" )
         "while" = @( "while ( CONDITION )", "{", "}" )
         "do .. while" = @( "do" , "{", "}", "while ( CONDITION )" )
         "do .. until" = @( "do" , "{", "}", "until ( CONDITION )" )
         "try" = @( "try", "{", "}" )
         "catch" = @( "catch", "{", "}" )
         "catch [<error type>] " = @( "catch [ERROR_TYPE]", "{", "}" )
         "finaly" = @( "finaly", "{", "}" )
     }
     
     Write-Host "Select snippet:"
     Write-Host
     $i = 1
     $snippetIndex = @()
     foreach ( $snippetName in $snippets.Keys | Sort )
     {
         Write-Host ( "{0} - {1}" -f $i++, $snippetName)
         $snippetIndex += $snippetName
     }
     try
     {
         [int]$choice = Read-Host ("Select 1-{0}" -f $i)
         if ( ( $choice -gt 0 ) -and ( $choice -lt $i ) )
         {
             $snippetName = $snippetIndex[$choice -1]
             
             Add-SnippetToEditor $snippets[$snippetName]
         }
         else
         {
             Throw "Choice not in range"
         }
     }
     catch
     {
         Write-Error "Choice was not in range or not even and integer"
     }
 }
 
 ##############################################################################################################
 ##############################################################################################################
 function Add-SnippetToEditor
 {
     param `
     (
         [string[]] $snippet
     )
 
     $editor = $psISE.CurrentOpenedFile.Editor
     $caretLine = $editor.CaretLine
     $caretColumn = $editor.CaretColumn
     $text = $editor.Text.Split("`n")
     $newText = @()
     if ( $caretLine -gt 1 )
     {
         $newText += $text[0..($caretLine -2)]
     }
     $curLine = $text[$caretLine -1]
     $indent = $curline -replace "[^\t ]", ""
     foreach ( $snipLine in $snippet )
     {
         $newText += $indent + $snipLine
     }
     if ( $caretLine -ne $text.Count )
     {
         $newText += $text[$caretLine..($text.Count -1)]
     }
     $editor.Text = [String]::Join("`n", $newText)
     $editor.SetCaretPosition($caretLine, $caretColumn)
 }
 
 Export-ModuleMember Add-Snippet
 
 
 ##############################################################################################################
 ##############################################################################################################
 if (-not( $psISE.CustomMenu.Submenus | where { $_.DisplayName -eq "Snippet" } ) )
 {
     $null = $psISE.CustomMenu.Submenus.Add("_Snippet", {Add-Snippet}, "Ctrl+Alt+S")
 }
`

