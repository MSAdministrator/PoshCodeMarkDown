---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 970
Published Date: 
Archived Date: 2009-03-27t18
---

# scan vb6 code - 

## Description

this functions work together to read the code of a visual basic project into a powershell object, which can be easyly scanned for patterns, returning filename, function/subroutine  name und the matching line. great tool you got unmaintained vb6 projects to understand for fixing or migrating (perhaps into powershell ;-))

## Comments



## Usage



## TODO



## function

`get-vbproject`

## Code

`#
 #
 
  
 
 function Get-VBProject ($file)
 {
     pushd (Split-path $file)
     foreach ($line in (gc $file))
     {
         if ($line -match "Class=\S+;\s+(\S+.cls)")
         {
             gi $matches[1]
         }
         elseif ($line -match "Form=(\S+.frm)")
         {
             gi $matches[1]
         }
         elseif ($line -match "Module=\S+;\s*(\S+.BAS)")
         {
             gi $matches[1]
         }
     }
     popd
 
 }
 
 function Get-VBLines ($file)
 {
     $oFile = gi $file
     $SourceLineNumber = 0
     foreach ($line in (gc $file))
     {
         $SourceLineNumber++
         if ($continue)
         {
             $buffer += "`r`n" + $line 
             if ($line -notmatch '_$') 
             { 
                 $continue = $False
                 $o = New-Object object
 		        Add-Member -InputObject $o -MemberType NoteProperty -Name LineNumber -Value $LineNumber
 		        Add-Member -InputObject $o -MemberType NoteProperty -Name VBLine     -Value $buffer
 		        Add-Member -InputObject $o -MemberType NoteProperty -Name file       -Value $oFile.Name
 		        Add-Member -InputObject $o -MemberType NoteProperty -Name filepath   -Value $oFile.FullName
                 $o
             }
         }
         else
         {
             $buffer = $line 
             $LineNumber = $SourceLineNumber 
             if ($line -match '_$') 
             { 
                 $continue = $True
             }
             else
             {
                 $o = New-Object object
 		        Add-Member -InputObject $o -MemberType NoteProperty -Name LineNumber -Value $LineNumber
 		        Add-Member -InputObject $o -MemberType NoteProperty -Name VBLine     -Value $buffer
 		        Add-Member -InputObject $o -MemberType NoteProperty -Name file       -Value $oFile.Name
 		        Add-Member -InputObject $o -MemberType NoteProperty -Name filepath   -Value $oFile.FullName
                 $o
             }
         }
         
     }
 }
  
 function Get-VB_CodeObject ()
 {
     param (     
         [Parameter(Position = 0, Mandatory=$true, ValueFromPipeline=$True)]
         $file    
     )     
     PROCESS
     {
         $inObject = $False
         $lines = @()
         $comments = @()
         $inDeclareations = $True
         Get-VBLines $file  | %  {
             if ($inObject)
             {
                 if ($_.VBLine -match "^End $end")
                 {
                     $lines += $_.VBLine
                     $inObject = $False
                     if ($inDeclareations){
                         $o = New-Object object
         		        Add-Member -InputObject $o -MemberType NoteProperty -Name LineNumber -Value 1
         		        Add-Member -InputObject $o -MemberType NoteProperty -Name Code       -Value $comments
         		        Add-Member -InputObject $o -MemberType NoteProperty -Name Comments   -Value ''
 
         		        Add-Member -InputObject $o -MemberType NoteProperty -Name file       -Value $_.file
         		        Add-Member -InputObject $o -MemberType NoteProperty -Name filepath   -Value $_.filepath
 
         		        Add-Member -InputObject $o -MemberType NoteProperty -Name acces      -Value ''
         		        Add-Member -InputObject $o -MemberType NoteProperty -Name typ        -Value ''
         		        Add-Member -InputObject $o -MemberType NoteProperty -Name name       -Value 'Declarations'
                         $inDeclareations = $false
                         $o
                         $comments = @()
                     }
 
                     $o = New-Object object
     		        Add-Member -InputObject $o -MemberType NoteProperty -Name LineNumber -Value $LineNumber
     		        Add-Member -InputObject $o -MemberType NoteProperty -Name Code       -Value $lines
        		        Add-Member -InputObject $o -MemberType NoteProperty -Name Comments   -Value $comments
                   
                     
     		        Add-Member -InputObject $o -MemberType NoteProperty -Name file       -Value $_.file
     		        Add-Member -InputObject $o -MemberType NoteProperty -Name filepath   -Value $_.filepath
 
     		        Add-Member -InputObject $o -MemberType NoteProperty -Name acces      -Value $acces
     		        Add-Member -InputObject $o -MemberType NoteProperty -Name typ        -Value $typ
     		        Add-Member -InputObject $o -MemberType NoteProperty -Name name       -Value $name
                     $o
                     $lines = @()
                 }
                 else
                     $lines += $_.VBLine
                 }
             }
             else
             {     
                 if ($_.VBLine -match "^(Public|Private)?\s*Property\s+(Get|Set)\s+([^\s\(]+)")
                 {
                     $acces = $matches[1]
                     $typ = 'Property ' + $matches[2]
                     $name = $matches[3]
                     $end =  'Property'
                     $inObject = $True
                     #$acces + " " +  $typ + " " + $name
                     $comments = $lines
                     $lines = (, $_.VBLine)
                     $LineNumber = $_.LineNumber
                     
                 }
                 elseif ($_.VBLine -match "^(Public|Private)?\s*(Function|Sub)\s+([^\s\(]+)")
                 {
                     $acces = $matches[1]
                     $typ = $matches[2]
                     $name = $matches[3]
                     $end = $matches[2]
                     $inObject = $True
                     #$acces + " " +  $typ + " " + $name
                     $comments = $lines
                     $lines = (, $_.VBLine)
                     $LineNumber = $_.LineNumber
                 }
                 else
                 {
                     $lines += $_.VBLine
                 }
             }
         }
     }
 }
 
 
 filter Select-VBCode_String ($pattern)
 {
     $o = New-Object object
     Add-Member -InputObject $o -MemberType NoteProperty -Name LineNumber -Value $_.LineNumber
    	Add-Member -InputObject $o -MemberType NoteProperty -Name Comments   -Value $_.comments
     
     Add-Member -InputObject $o -MemberType NoteProperty -Name file       -Value $_.file
     Add-Member -InputObject $o -MemberType NoteProperty -Name filepath   -Value $_.filepath
 
 	Add-Member -InputObject $o -MemberType NoteProperty -Name acces      -Value $_.acces
     Add-Member -InputObject $o -MemberType NoteProperty -Name typ        -Value $_.typ
 	Add-Member -InputObject $o -MemberType NoteProperty -Name name       -Value $_.name
 
     $patternFound = $False
     $lines = @() 
     foreach ($l in $_.Code)
     {
         if ($l -match $pattern)
         {
             $lines += $l
             $patternFound = $True
         }
     }
     if ( $patternFound)
     {
         Add-Member -InputObject $o -MemberType NoteProperty -Name Code       -Value $lines
         $o
     }
 
 }
 
 
 <#
     $myVBProject = 'myDirtyVb6Project.vbp'
     
     
     $vbp = Get-VBProject $myVBProject | Get-VB_CodeObject 
 
     
     $vbp | Select-VBCode_String('"select') |% { $_.code }
 
     $vbp| Select-VBCode_String('EXIT') | fl
 
     $vbp| select file, acces, typ, name
 
 #>
`

