---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 3.0
Encoding: ascii
License: cc0
PoshCode ID: 5531
Published Date: 2014-10-21t07
Archived Date: 2014-12-24t21
---

# edit-code 3 - 

## Description

opens a function in a text editor….

## Comments



## Usage



## TODO



## function

`split-command`

## Code

`#
 #
 
 
 function Split-Command {
     param(
         [Parameter(Mandatory=$true)]
         [string]$Command
     )
     $Parts = @($Command -Split " ")
 
     for($count=$Parts.Length; $count -gt 1; $count--) {
         $Editor = ($Parts[0..$count] -join " ").Trim("'",'"')
         if((Test-Path $Editor) -and (Get-Command $Editor -ErrorAction SilentlyContinue)) { 
             $Editor
             $Parts[$($Count+1)..$($Parts.Length)] -join " "
             break
         }
     }
 }
 
 function Find-Editor {
     #.Synopsis
     #.Description
     [CmdletBinding()]
     param
     (
         [Parameter(Position=1)] 
         [System.String]
         $Editor = $(if($global:PSEditor.Command){ $global:PSEditor.Command } else { $global:PSEditor } ),
 
         [Parameter(Position=2)]
         [System.String]
         $Parameters = $global:PSEditor.Parameters
     )
     
     end {
             if($Editor -and (Get-Command $Editor)) { break }
 
             if ($Editor -and !(Get-Command $Editor -ErrorAction SilentlyContinue))
             {
                 Write-Verbose "Editor is not a valid command, split it:"
                 $Editor, $Parameters = Split-Command $Editor
                 if($Editor) { break }
             }
 
             if (Test-Path Env:Editor)
             {
                 Write-Verbose "Editor was not passed in, trying Env:Editor"
                 $Editor, $Parameters = Split-Command $Env:Editor
                 if($Editor) { break }
             }
 
             if (Get-Command Git -ErrorAction SilentlyContinue)
             {
                 Write-Verbose "PSEditor and Env:Editor not found, searching git config"
                 $Editor, $Parameters = Split-Command (git config core.editor)
                 if($Editor) { break }
             }
 
             Write-Verbose "Editor not found, trying some others"
             if($Editor = Get-Command "subl.exe", "sublime_text.exe" | Select-Object -Expand Path -first 1)
             {
                 $Parameters = "-n -w"
                 break
             }
             if($Editor = Get-Command "notepad++.exe" | Select-Object -Expand Path -first 1)
             {
                 $Parameters = "-multiInst"
                 break
             }
             if($Editor = Get-Command "atom.exe" | Select-Object -Expand Path -first 1)
             {
                 break
             }
 
             Write-Verbose "Editor still not found, getting desperate:"
             if(($Editor = Get-Item "C:\Program Files\DevTools\Sublime Text ?\sublime_text.exe" -ErrorAction SilentlyContinue | Sort {$_.VersionInfo.FileVersion} -Descending | Select-Object -First 1) -or
                ($Editor = Get-ChildItem C:\Program*\* -recurse -filter "sublime_text.exe" -ErrorAction SilentlyContinue | Select-Object -First 1))
             {
                 $Parameters = "-n -w"
                 break
             }
 
             if(($Editor = Get-ChildItem "C:\Program Files\Notepad++\notepad++.exe" -recurse -filter "notepad++.exe" -ErrorAction SilentlyContinue | Select-Object -First 1) -or
                ($Editor = Get-ChildItem C:\Program*\* -recurse -filter "notepad++.exe" -ErrorAction SilentlyContinue | Select-Object -First 1))
             {
                 $Parameters = "-multiInst"
                 break
             }
 
             Write-Verbose "Editor not found, settling for notepad"
             $Editor = "notepad"
 
             if(!$Editor -or !(Get-Command $Editor -ErrorAction SilentlyContinue -ErrorVariable NotFound)) { 
                 if($NotFound) { $PSCmdlet.ThrowTerminatingError( $NotFound[0] ) }
                 else { 
                     throw "Could not find an editor (not even notepad!)"
                 }
             }
         } while($false)
 
         $PSEditor = New-Object PSObject -Property @{
                 Command = "$Editor"
                 Parameters = "$Parameters"
         } | Add-Member ScriptMethod ToString -Value { "'" + $this.Command + "' " + $this.Parameters } -Force -PassThru
 
         if($PSBoundParameters.ContainsKey("Editor") -or 
            $PSBoundParameters.ContainsKey("Parameters") -or 
            !(Test-Path variable:global:PSeditor) -or 
            ($PSEditor.Command -ne $Editor))
         {
             Write-Verbose "Setting global preference variable for Editor: PSEditor"
             $global:PSEditor = $PSEditor
 
             if(![Environment]::GetEnvironmentVariable("Editor", "User")) {
                 Write-Verbose "Setting user environment variable: Editor"
                 [Environment]::SetEnvironmentVariable("Editor", "$PSEditor", "User")
             }
         }
         return $PSEditor
     }
 }
 
 function Edit-Code {
     <#
         .SYNOPSIS
             Creates and edits functions (or scripts) in the session in a script editor.
 
         
         .DESCRIPTION
             The Edit-Function command lets you create or edit functions in your session in your favorite text editor.
 
             It opens the specified function in the editor that you specify, and when you finish editing the function and close the editor, the script updates the function in your session with the new function code.
 
             Functions are tricky to edit, because most code editors require a file, and determine syntax highlighting based on the extension of that file. Edit-Function creates a temporary file with the function code.
 
             If you have a favorite editor, you can use the Editor parameter to specify it once, and the script will save it as your preference. If you don't specify an editor, it tries to determine an editor using the PSEditor preference variable, the EDITOR environment variable, or your configuration for git.  As a fallback it searches for Sublime, and finally falls back to Notepad. 
 
             REMEMBER: Because functions are specific to a session, your function edits are lost when you close the session unless you save them in a permanent file, such as your Windows PowerShell profile.
         
         .EXAMPLE
             Edit-Function Prompt
 
             Opens the prompt function in a default editor (gitpad, Sublime, Notepad, whatever)
 
         .EXAMPLE
             dir Function:\cd* | Edit-Function -Editor "C:\Program Files\Sublime Text 3\subl.exe" -Param "-n -w"
 
             Pipes all functions starting with cd to Edit-Function, which opens them one at a time in a new sublime window (opens each one after the other closes).
 
         .EXAMPLE
             Get-Command TabExpan* | Edit-Function -Editor 'C:\Program Files\SAPIEN Technologies, Inc\PowerShell Studio 2014\PowerShell Studio.exe
             
             Edits the TabExpansion and/or TabExpansion2 (whichever exists) in PowerShell Studio 2014 using the full path to the .exe file.
             Note that this also sets PowerShell Studio as your default editor for future calls.
 
         .NOTES
             By Joel Bennett (@Jaykul) and June Blender (@juneb_get_help)
             
             If you'd like anything changed  ... feel free to push new version on PoshCode, or tweet at me :-)
             - Do you not like that I make every editor the default?
             - Think I should detect notepad2 or notepad++ or something?
             
             About ISE: it doesn't support waiting for the editor to close, sorry.
     #>
     [CmdletBinding(DefaultParameterSetName="Command")]
     param (
         [Parameter(Position=0, Mandatory = $true, ValueFromPipelineByPropertyName = $true, ParameterSetName="Command")]
         [String]
         $Name,
 
         [Parameter(Mandatory = $true, ValueFromPipelineByPropertyName = $true, ParameterSetName="File")]
         [Alias("PSPath")]
         [String[]]
         $Path,
 
         [Parameter(Position=1)] 
         [String]
         $Editor = $(if($global:PSEditor.Command){ $global:PSEditor.Command } else { $global:PSEditor } ),
 
         [Parameter(Position=2)]
         [String]
         $Parameters = $global:PSEditor.Parameters,
 
         [Switch]$NoWait,
 
         [Switch]$Force
     )
     begin {
         $TextApplications  = ".cmd",".bat",".vbs",".pl",".rb",".py",".wsf",".js"
         $NonFileCharacters = "[$(([IO.Path]::GetInvalidFileNameChars() | %{ [regex]::escape($_) }) -join '|')]"
 
         $RejectAll = $false;
         $ConfirmAll = $false;
     }
     process {
         [String[]]$Files = @()
         $MaxDepth = 10
         if($PSCmdlet.ParameterSetName -eq "Command") {
             while($Command = Get-Command $Name -Type Alias -ErrorAction SilentlyContinue) {
                 $Name = $Command.definition
                 if(($MaxDepth--) -lt 0) { break }
             }
 
             $Files = @(
                 switch($Command = Get-Command $Name -ErrorAction "SilentlyContinue" -Type "Function", "ExternalScript", "Application" | Select-Object -First 1) {
                     { $_.CommandType -eq "Function"}{
                         Write-Verbose "Found a function matching $Name"
                         $File = [IO.Path]::GetTempFileName() | 
                             Rename-Item -NewName { [IO.Path]::ChangeExtension($_, ".tmp.ps1") } -PassThru |
                             Select-Object -Expand FullName
 
                         if (Test-Path Function:\$Name) {
                             Set-Content -Path $File -Value $((Get-Content Function:\$Name) -Join "`n")
                         }
                         $File
                     }
 
                     {$_.CommandType -eq "ExternalScript"}{
                         Write-Verbose "Found an ExternalScript matching $Name"
                         $_.Path
                     }
 
                     {$_.CommandType -eq "Application"} {
                         Write-Verbose "Found an Application or Script matching $Name"
                         if(($TextApplications -contains $_.Extension) -or $Force -Or $PSCmdlet.ShouldContinue("Are you sure you want to edit '$($_.Path)' in a text editor?", "Opening '$($_.Name)'", [ref]$ConfirmAll, [ref]$RejectAll)) {
                             $_.Path
                         }
                     }
                 }
             )
 
             if($Files.Length -eq 0) {
                 Write-Verbose "No '$Name' command found, resolving file path"
                 $Files = @(Resolve-Path $Name -ErrorAction SilentlyContinue | Select-Object -Expand Path)
 
                 if($Files.Length -eq 0) {
                     Write-Verbose "Still no file found, they're probably trying to create a new function"
                     if($Name -notmatch $NonFileCharacters) {
                         $File = [IO.Path]::GetTempFileName() | 
                             Rename-Item -NewName { [IO.Path]::ChangeExtension($_, ".tmp.ps1") } -PassThru |
                             Select-Object -Expand FullName
 
                         if (Test-Path Function:\$Name) {
                             Set-Content -Path $file -Value $((Get-Content Function:\$Name) -Join "`n")
                         }
                         $Files = @($File)
                     }
                 } else {
                     $Files
                 }
             }
         } else {
             Write-Verbose "Resolving file path, because although we'll create files, we won't create directories"
             $Folder = Split-Path $Path
             $FileName = Split-Path $Path -Leaf
             $Files = @(
                 if($Folder -and -not (Resolve-Path $Folder -ErrorAction SilentlyContinue)) {
                     Write-Error "The path '$Folder' doesn't exist, so we cannot create '$FileName' there"
                     return
                 } elseif($FileName -notmatch $NonFileCharacters) {
                     foreach($F in Resolve-Path $Folder -ErrorAction SilentlyContinue) {
                         Join-Path $F $FileName
                     }
                 } else {
                     Resolve-Path $Path -ErrorAction SilentlyContinue | Select-Object -Expand Path
                 }
             )
         }
 
         $PSEditor = Find-Editor 
 
         foreach($File in @($Files)) {
             if($File) {
                 $LastWriteTime = (Get-Item $File).LastWriteTime
 
                 Write-Verbose "$PSEditor '$File'"
                 if ($PSEditor.Parameters)
                 {
                     Start-Process -FilePath $PSEditor.Command -ArgumentList $PSEditor.Parameters, $file -Wait:(!$NoWait)
                 }
                 else
                 {
                     Start-Process -FilePath $PSEditor.Command -ArgumentList $file -Wait:(!$NoWait)
                 }
                 
                 if($File.EndsWith(".tmp.ps1") -and $File.StartsWith(([IO.Path]::GetTempPath()))) {
 
                     if($LastWriteTime -ne (Get-Item $File).LastWriteTime) {
                         Write-Verbose "Changed $Name function"
                         Set-Content -Path Function:\$Name -Value ([scriptblock]::create(((Get-Content $file) -Join "`n")))
                     } else {
                         Write-Warning "No change to $Name function"
                     }
 
                     Write-Verbose "Deleting temp file $File"
                     Remove-Item $File
                 }
             }
         }
     }
 }
 
 Set-Alias edit Edit-Code
 Export-ModuleMember -Function Edit-Code, Find-Editor -Alias edit
`

