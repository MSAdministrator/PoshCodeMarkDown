---
Author: brandon warner
Publisher: 
Copyright: 
Email: 
Version: 2016.06.23
Encoding: ascii
License: cc0
PoshCode ID: 6422
Published Date: 2016-06-28t15
Archived Date: 2016-07-01t08
---

# tsql auto-programming - 

## Description

this function is simple example of using smo to iterate through some database objects, tables

## Comments



## Usage



## TODO



## script

`get-wordwrappedtext`

## Code

`#
 #
 <#+----------------------------------------------------------------------------------------------+#
   | Load Assemblies                                                                              |
  #+----------------------------------------------------------------------------------------------+#>
 [System.Reflection.Assembly]::LoadWithPartialName('Microsoft.SqlServer.Smo')          | Out-Null
 [System.Reflection.Assembly]::LoadWithPartialName('Microsoft.SqlServer.SmoExtended')  | Out-Null
 
 <#
 .SYNOPSIS
 Takes a text string and word-wraps it to a specific length. 
 
 .DESCRIPTION
 This function takes a string of text and word-wraps it to a specific target line width. In addition 
 to word-wrapping the applies optionally prefix and suffix to each line, and also an optional 
 indent based on tab with and indent level.
 
 .NOTES
 +--------------------------------------------------------------------------------------------------+
 | ORIGIN STORY                                                                                     |
 +--------------------------------------------------------------------------------------------------+
 |   DATE        : 2016-06-23
 |   AUTHOR      : Brandon Warner
 |   DESCRIPTION : Initial Draft
 +--------------------------------------------------------------------------------------------------+
 |   DATE        : 2016-06-27
 |   AUTHOR      : Brandon Warner
 |   DESCRIPTION : Fixed bug with line wrapping for lines less than the wrap-width
                   Also removed line endings from the last line of the returned wrapped text
 +--------------------------------------------------------------------------------------------------+
 
 .PARAMETER Text
 The source text that we want to word-wrap.
 
 .PARAMETER WrapWidth
 The target width of one line in the wrapped text.
 
 .PARAMETER Prefix
 A text prefix that should be appended to the beginning of each line in resulting wrapped text
 (useful for box drawing)
 
 .PARAMETER Suffix
 A text suffix that should be appended to the end of each line in resulting wrapped text
 (useful for box drawing)
 
 .PARAMETER TabWidth
 Defines the character length of each indent level, if a non-zero indent level is supplied.
 
 .PARAMETER IndentLevel
 Indicates the number of levels to indent the resulting wrapped text.
 
 .EXAMPLE
 
 $ThisText  = 'Hello this is a really, really, really, really long sentence that I want to be word-wrapped'
 $ThisText += ' so that it displays nicely in a document (either printed or on screen) so that I avoid having to scroll'
 $ThisText += ' back and forth, back and forth, wasting both time and effort, that will not have to be spent,'
 $ThisText += ' when the text is properly formatted.'
 
 Get-WordWrappedText `
   -Text $ThisText `
   -WrapWidth 50 `
   -Prefix '>' `
   -IndentLevel 2 | oh
 
 #>
 function Get-WordWrappedText 
   {
     [CmdletBinding()]
     Param
       (
           [Parameter(Mandatory = $true)]
           [string]$Text  
 
         , [Parameter(Mandatory = $false)]
           [ValidateRange(25,150)]
           [Int]$WrapWidth = 100
 
         , [Parameter(Mandatory = $false)]
           [ValidateLength(1,25)]  
           [String]$Prefix = ''
   
         , [Parameter(Mandatory = $false)]
           [ValidateLength(1,25)]
           [String]$Suffix = ''  
   
         , [Parameter(Mandatory = $false)]
           [ValidateRange(1,8)]
           [Int]$TabWidth = 2
   
         , [Parameter(Mandatory = $false)]
           [ValidateRange(0,75)]
           [Int]$IndentLevel = 0    
       )
   
     if(($WrapWidth - "$Prefix$Suffix".Length) -lt 10)
       { 
         Write-Error `
           -Message "The wrap width (minus prefix and suffix length) is too short. There should be at least 10 characters width of text space left over after the prefix and suffix." 
       }
   
     [String]$Result       = ''
     [String]$ThisLine     = ''
     [String]$IndentSpace  = ''.PadLeft($TabWidth*$IndentLevel)
    
     <#+--------------------------------------------------------------------+#\
       | See if the whole thing fits on one line                            |
     \#+--------------------------------------------------------------------+#>
     [Int]$LineWidth     = "$Prefix $Text $Suffix".Length
     
     if($LineWidth -le $WrapWidth)
       {
         return "$IndentSpace$Prefix $Text$("$Suffix".PadLeft($WrapWidth - $Text.Length - $Prefix.Length))"
       }
   
     <#+--------------------------------------------------------------------+#\
       | Proceed with word-wrapping the text                                |
     \#+--------------------------------------------------------------------+#>  
     else
       {
         [Bool]$Done         = $false
         [Int]$Position      = 0
         [Int]$PositionPrev  = 0
 
         while(!$Done)
           {
             $PositionPrev = $Position 
             $Position += $WorkingWidth
    
             if($Position -lt $Text.Length)
               {        
                 <#+-------------------------------------------+#\
                   | Inch back till you hit a space            |
                 \#+-------------------------------------------+#>  
                 while(!($Text.Substring($Position,1) -match '\s') -and ($Position -gt $PositionPrev))
                   { 
                     $Position-- 
                   }
               }
             else
               {
                 <#+--------------------------------------------------------------------+#\
                   | If we over-shot, inch back to the end of the text                  |
                 \#+--------------------------------------------------------------------+#>
                 while($Position -ge $Text.Length)
                   { 
                     $Position-- 
                   }              
                 $Done = $true
               }
       
             <#+------------------------------------------------------------------------------+#\
               | If there entire line width was unbroken by white space, break the word       |
             \#+------------------------------------------------------------------------------+#>
             if($Position -eq $PositionPrev)
               {
                 $Position += $WorkingWidth 
               }
       
             $LineWidth = $Position - $PositionPrev 
 
             $Result += "$IndentSpace$Prefix $($Text.Substring($PositionPrev,$LineWidth))$(" $Suffix".PadLeft($WrapWidth-$LineWidth-$Prefix.Length))"
             if(!$Done)
               {
                 $Result += "`r`n"
               }
       
             <#+--------------------------------------------------------------------+#\
               | Inch past white space to the next word                             |
             \#+--------------------------------------------------------------------+#>      
             while(($Text.Substring($Position,1) -match '\s') -and ($Position -lt $Text.Length))
               { 
                 $Position++ 
               }
           }   
       }
     return $Result
   }
 
 <#
 .SYNOPSIS
 Returns comment block of word-wrapped text inside a character based box drawing.
 
 .DESCRIPTION
 Given a string of body text which you want to appear in a comment block surrounded in box
 drawing characters, produces said comment block, word-wrapped to a given width and indented
 to a given level as per the indent level and tab width.
 
 .NOTES
 +--------------------------------------------------------------------------------------------------+
 | ORIGIN STORY                                                                                     |
 +--------------------------------------------------------------------------------------------------+
 |   DATE        : 2016-06-23
 |   AUTHOR      : Brandon Warner
 |   DESCRIPTION : Initial Draft
 +--------------------------------------------------------------------------------------------------+
 |   DATE        : 2016-06-27
 |   AUTHOR      : Brandon Warner
 |   DESCRIPTION : Adjusted final return string to account for change in line endings of last line
                   returned in Get-WordWrappedText
 +--------------------------------------------------------------------------------------------------+
 
 .PARAMETER CommentText
 The text that we want to appear in the comment block.
 
 .PARAMETER SectionWidth
 The target width of the comment block
 
 .PARAMETER TabWidth
 The number of characters which the comment block will be indented for each indentation level.
 
 .PARAMETER IndentLevel
 The number of indentation levels which we want to indent the text by.
 
 .PARAMETER Encoding
 'Unicode' for special Unicode box-drawing characters to be used. 'OEM' for approximate ANSI characters to appear instead.
 
 .PARAMETER CommentStyle
 Specifies the comment style for the programming language we are providing the comment block for.
 
 .EXAMPLE
 
 Get-BoxedComment `
   -CommentText 'Line breaking, also known as word wrapping, is the process of breaking a section of text into lines such that it will fit in the available width of a page, window or other display area. In text display, line wrap is the feature of continuing on a new line when a line is full, such that each line fits in the viewable window, allowing text to be read from top to bottom without any horizontal scrolling. Word wrap is the additional feature of most text editors, word processors, and web browsers, of breaking lines between words rather than within words, when possible. Word wrap makes it unnecessary to hard-code newline delimiters within paragraphs, and allows the display of text to adapt flexibly and dynamically to displays of varying sizes.' `
   -IndentLevel 2 `
   -CommentStyle PowerShell | Out-File -FilePath "$env:USERPROFILE\CommentBlockSample.ps1" -Force
 
 #>
 
 function Get-BoxedComment
   {
     [OutputType([String])]    
     [CmdletBinding()]
     Param
       (
           [Parameter(Mandatory = $true)]
           [string]$CommentText  
 
         , [Parameter(Mandatory = $false)]
           [ValidateRange(25,150)]
           [Int]$SectionWidth = 100
 
         , [Parameter(Mandatory = $false)]
           [ValidateRange(1,8)]
           [Int]$TabWidth = 2
   
         , [Parameter(Mandatory = $false)]
           [ValidateRange(0,50)]
           [Int]$IndentLevel = 0  
 
         , [Parameter(Mandatory = $false)]
           [ValidateSet('Unicode','OEM')]
           [String]$Encoding = 'Unicode'  
 
         , [Parameter(Mandatory = $false)]
           [ValidateSet('SQL','PowerShell')]
           [String]$CommentStyle = 'SQL'  
       )
 
     <#+--------------------------------------------------------------------+#\
       | Define the box drawing characters                                  |
     \#+--------------------------------------------------------------------+#>  
     switch($Encoding)
       {
         'Unicode' 
           {
             $BoxHorizontalLineChar     = '-'
             $BoxVerticalLineChar       = '|'
             $BoxUpperLeftCornerChar    = '+'
             $BoxUpperRightCornerChar   = '+'
             $BoxLowerLeftCornerChar    = '+'
             $BoxLowerRightCornerChar   = '+'
           }
         'OEM'
           {
             $BoxHorizontalLineChar     = '-'
             $BoxVerticalLineChar       = '|'
             $BoxUpperLeftCornerChar    = '+'
             $BoxUpperRightCornerChar   = '+'
             $BoxLowerLeftCornerChar    = '+'
             $BoxLowerRightCornerChar   = '+'
           }
       }
   
     <#+--------------------------------------------------------------------+#\
       | Define the multi-line commenting delimiters & corresponding        |
       | symmetry embellishments as per the given comment style             |  
     \#+--------------------------------------------------------------------+#>
     switch($CommentStyle)
       {
         'SQL' 
           {
             $MultilineCommentOpen   = '/*'
             $UpperRightEmbelishment = '*\'
             $LowerLeftEmbelishment  = '\*'
             $MultilineCommentClose  = '*/'
           }
         'PowerShell'
           {
             $MultilineCommentOpen   = '<#'
             $LowerLeftEmbelishment  = ' #'
             $MultilineCommentClose  = '#>' 
           }
       }  
 
     [String]$BoxHorizontalLine  = $BoxHorizontalLineChar.PadRight($SectionWidth-6).Replace(' ',$BoxHorizontalLineChar)
     [String]$IndentSpace        = ''.PadLeft($TabWidth*$IndentLevel)
     [String]$Body               = Get-WordWrappedText `
                                     -Text $CommentText `
                                     -Prefix "$("$BoxVerticalLineChar".PadLeft($MultilineCommentOpen.Length + 1))" `
                                     -Suffix "$BoxVerticalLineChar" `
                                     -WrapWidth ($SectionWidth - $MultilineCommentOpen.Length -1 ) `
                                     -TabWidth $TabWidth `
                                     -IndentLevel $IndentLevel
   
     return @"
 $($IndentSpace)$($MultilineCommentOpen)$($BoxUpperLeftCornerChar)$($BoxHorizontalLine)$($BoxUpperRightCornerChar)$($UpperRightEmbelishment)
 $($Body)
 $($IndentSpace)$($LowerLeftEmbelishment)$($BoxLowerLeftCornerChar)$($BoxHorizontalLine)$($BoxLowerRightCornerChar)$($MultilineCommentClose)
   
 "@
   }
 
 <#
 .SYNOPSIS
 Example for Automatic Programming of Transact-SQL code using SQL Server Management Objects (SMO)
 
 .DESCRIPTION
 This function is simple example of using SMO to iterate through some database objects, tables 
 and table rows, in this case, in order to generate a T-SQL script. In this example, I have 
 created a script that creates a view for each table that selects the top 100 rows of table. 
 Though I doubt this would be very useful, this is simply to exemplify the technique of 
 looping through the SMO API to generate code. This can be extremely powerful in generating 
 a variety of scripts based on property values available in this API.
 
 .NOTES
 +----------------------------------------------------------------------------------------------+
 | REVISION HISTORY                                                                             |
 +----------------------------------------------------------------------------------------------+
 |   DATE        : 2016.06.23
 |   AUTHOR      : Brandon Warner
 |   DESCRIPTION : Initial Draft
 +----------------------------------------------------------------------------------------------+
 
 .PARAMETER ServerInstance
 Target SQL Server\Instance
 
 .PARAMETER Database
 Target Database
 
 .PARAMETER Table
 Target Table
 
 .PARAMETER DestinationScriptPath
 Destination File Path for the script we want to write
 
 .EXAMPLE 
 
 Write-TsqlScript `
   -ServerInstance 'MyServer\MyInstance,MyListenerPort' `
   -Database 'MyDatabase' `
   -DestinationScriptPath "$env:USERPROFILE\AutomaticProgrammingTSQL_SMO_Example.sql"
 
 #>
 function Write-TsqlScript
   {
     [CmdletBinding()]
     Param
       (
           [Parameter(Mandatory=$true)]
           [String]$ServerInstance
   
         , [Parameter(Mandatory=$true)]
           [String]$Database
 
         , [Parameter(Mandatory=$false)]
           [String]$DestinationScriptPath  
       )
 
     <#+--------------------------------------------------------------------+#
       | Connect to SQL Server via SMO                                      |
      #+--------------------------------------------------------------------+#>
     $SmoServer = New-Object Microsoft.SqlServer.Management.Smo.Server $ServerInstance
     $SmoServer.ConnectionContext.Disconnect() | Out-Null
     $SmoServer.ConnectionContext.ApplicationName = 'PowerShell Script'
     $SmoServer.ConnectionContext.LoginSecure = $true
     $SmoServer.ConnectionContext.Connect()
 
     [String]$ViewName = "dbo.vw_$($Table)_Top100"
     [String]$TSQL     = @"
 /*+---------------------------------------------------------------------------------------------+*\
   | TITLE: $("Create `"Top 100`" Views for user tables in database $Database.".PadRight(85))|
   +---------------------------------------------------------------------------------------------+
   | DESCRIPTION:                                                                                |
   |                                                                                             |
   |   Initially Auto-Generated by PowerShell Script:                                            |
 $(Get-WordWrappedText `
   -Text ($MyInvocation.ScriptName) `
   -WrapWidth 94 `
   -Prefix '|    ' `
   -Suffix '|' `
   -IndentLevel 1)
   |   Function:                                                                                 |
 $(Get-WordWrappedText `
   -Text ($MyInvocation.MyCommand.Name) `
   -WrapWidth 94 `
   -Prefix '|    ' `
   -Suffix '|' `
   -IndentLevel 1)  
   +---------------------------------------------------------------------------------------------+
   | REVISION HISTORY:                                                                           |
   +---------------------------------------------------------------------------------------------+
   |   DATE       AUTHOR          CHANGE DESCRIPTION                                             |
   |   ---------- --------------- ---------------------------------------------------------------+
   |   $((Get-Date).ToString('yyyy.MM.dd')) $($env:USERNAME.PadRight(15)) Initial Draft$("".PadLeft(50))|
 \*+---------------------------------------------------------------------------------------------+*/
 
 "@
   
     <#+----------------------------------------------------------------------------------------------+#
       | Loop through all the non-system tables in the given database                                 |
      #+----------------------------------------------------------------------------------------------+#>
     $TargetTables = $SmoServer.Databases[$Database].Tables | where {!$_.IsSystemObject} 
     $TableCount = $TargetTables.Count
   
     [Int]$i = 1
     $TargetTables |
       % {
           $Table = $_.Name
           
           Write-Progress -Activity 'Generating Auto-Programmed TSQL Script' -Status "Scripting for table $Table" -PercentComplete ([Math]::Ceiling(($i/$TableCount)*100.00))
     
           <#+----------------------------------------------------------------------------------------------+#
             | Create comment header-block for this script & other setup before looping through columns     |
            #+----------------------------------------------------------------------------------------------+#>
           [String]$ViewName = "dbo.vw_$($Table)_Top100"
           $TSQL += @"
 
 $(Get-BoxedComment `
   -CommentText "Create View: $ViewName" `
   -IndentLevel 1 `
   -SectionWidth 75 `
   -CommentStyle SQL)
   IF OBJECT_ID('$ViewName','view') IS NOT NULL
    DROP VIEW $ViewName
   GO
 
   CREATE VIEW
     $ViewName
   AS
   SELECT TOP 100
 "@
           [Bool]$IsFirst    = $true
           [Int]$PadLength   = 0
           [Char]$TableAlias = $Table.Substring(0,1).ToLower()
         
           <#+----------------------------------------------------------------------------------------------+#
             | Find the longest column name for code alignment purposes                                     |
            #+----------------------------------------------------------------------------------------------+#>  
           $_.Columns | where {!$_.IsSystemObject} | 
             % { 
                 if($_.Name.Length -gt $PadLength) 
                   { 
                     $PadLength = $_.Name.Length 
                   } 
               }
           $PadLength += 2
           if($PadLength % 2 -eq 1) 
             { 
               $PadLength += 1 
             }
         
           <#+----------------------------------------------------------------------------------------------+#
             | Loop through the columns in the table, adding each to the select statement                   |
            #+----------------------------------------------------------------------------------------------+#>  
           $_.Columns | where {!$_.IsSystemObject} | 
             % {
                 if($IsFirst)
                   {
                     $TSQL += @"
 
       $($_.Name.PadRight($PadLength))= $TableAlias.$($_.Name)
 
 "@
                   }
                 else
                   {
                     $TSQL += @"
     , $($_.Name.PadRight($PadLength))= $TableAlias.$($_.Name)
 
 "@
                   }
                 $IsFirst = $false          
               }
 
           <#+--------------------------------------------------------------------+#
             | Finalize the create view statement for this table                  |
            #+--------------------------------------------------------------------+#>
           $TSQL += @"
   FROM
     dbo.$Table $TableAlias
   GO
 
 "@    
           $i++
         }
   
     <#+----------------------------------------------------------------------------------------------+#
       | Display the final script to the host terminal and write out the file if a destination file   |
       | path was given                                                                               |
      #+----------------------------------------------------------------------------------------------+#> 
     $TSQL | Out-Host  
     if($DestinationScriptPath)
       {
         $TSQL | Out-File -FilePath $DestinationScriptPath -Force
       }
   }
`

