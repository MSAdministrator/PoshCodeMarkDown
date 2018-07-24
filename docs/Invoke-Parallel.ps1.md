---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5873
Published Date: 
Archived Date: 2015-05-29t10
---

#  - 

## Description

trying to get failed wmi and successful wmi machines out into a variable… keep getting “positional parameter” errors when i try to create $wmicheckresults custom object (first time i’ve ever had to use a new object) ... ideally i don’t dump the results to file at all, but i can’t figure out how to get it out of the invoke-parallel scriptblock once it’s done it’s code..just disappears into the ether…

## Comments



## Usage



## TODO



## function

`invoke-parallel`

## Code

`#
 #
 function Invoke-Parallel {
     <#
     .SYNOPSIS
         Function to control parallel processing using runspaces
 
     .DESCRIPTION
         Function to control parallel processing using runspaces
 
             Note that each runspace will not have access to variables and commands loaded in your session or in other runspaces by default.  
             This behaviour can be changed with parameters.
 
     .PARAMETER ScriptFile
         File to run against all input objects.  Must include parameter to take in the input object, or use $args.  Optionally, include parameter to take in parameter.  Example: C:\script.ps1
 
     .PARAMETER ScriptBlock
         Scriptblock to run against all computers.
 
         You may use $Using:<Variable> language in PowerShell 3 and later.
         
             The parameter block is added for you, allowing behaviour similar to foreach-object:
                 Refer to the input object as $_.
                 Refer to the parameter parameter as $parameter
 
     .PARAMETER InputObject
         Run script against these specified objects.
 
     .PARAMETER Parameter
         This object is passed to every script block.  You can use it to pass information to the script block; for example, the path to a logging folder
         
             Reference this object as $parameter if using the scriptblock parameterset.
 
     .PARAMETER ImportVariables
         If specified, get user session variables and add them to the initial session state
 
     .PARAMETER ImportModules
         If specified, get loaded modules and pssnapins, add them to the initial session state
 
     .PARAMETER Throttle
         Maximum number of threads to run at a single time.
 
     .PARAMETER SleepTimer
         Milliseconds to sleep after checking for completed runspaces and in a few other spots.  I would not recommend dropping below 200 or increasing above 500
 
     .PARAMETER RunspaceTimeout
         Maximum time in seconds a single thread can run.  If execution of your code takes longer than this, it is disposed.  Default: 0 (seconds)
 
         WARNING:  Using this parameter requires that maxQueue be set to throttle (it will be by default) for accurate timing.  Details here:
         http://gallery.technet.microsoft.com/Run-Parallel-Parallel-377fd430
 
     .PARAMETER NoCloseOnTimeout
 		Do not dispose of timed out tasks or attempt to close the runspace if threads have timed out. This will prevent the script from hanging in certain situations where threads become non-responsive, at the expense of leaking memory within the PowerShell host.
 
     .PARAMETER MaxQueue
         Maximum number of powershell instances to add to runspace pool.  If this is higher than $throttle, $timeout will be inaccurate
         
         If this is equal or less than throttle, there will be a performance impact
 
         The default value is $throttle times 3, if $runspaceTimeout is not specified
         The default value is $throttle, if $runspaceTimeout is specified
 
     .PARAMETER LogFile
         Path to a file where we can log results, including run time for each thread, whether it completes, completes with errors, or times out.
 
 	.PARAMETER Quiet
 		Disable progress bar.
 
     .EXAMPLE
         Each example uses Test-ForPacs.ps1 which includes the following code:
             param($computer)
 
             if(test-connection $computer -count 1 -quiet -BufferSize 16){
                 $object = [pscustomobject] @{
                     Computer=$computer;
                     Available=1;
                     Kodak=$(
                         if((test-path "\\$computer\c$\users\public\desktop\Kodak Direct View Pacs.url") -or (test-path "\\$computer\c$\documents and settings\all users
 
         \desktop\Kodak Direct View Pacs.url") ){"1"}else{"0"}
                     )
                 }
             }
             else{
                 $object = [pscustomobject] @{
                     Computer=$computer;
                     Available=0;
                     Kodak="NA"
                 }
             }
 
             $object
 
     .EXAMPLE
         Invoke-Parallel -scriptfile C:\public\Test-ForPacs.ps1 -inputobject $(get-content C:\pcs.txt) -runspaceTimeout 10 -throttle 10
 
             Pulls list of PCs from C:\pcs.txt,
             Runs Test-ForPacs against each
             If any query takes longer than 10 seconds, it is disposed
             Only run 10 threads at a time
 
     .EXAMPLE
         Invoke-Parallel -scriptfile C:\public\Test-ForPacs.ps1 -inputobject c-is-ts-91, c-is-ts-95
 
             Runs against c-is-ts-91, c-is-ts-95 (-computername)
             Runs Test-ForPacs against each
 
     .EXAMPLE
         $stuff = [pscustomobject] @{
             ContentFile = "windows\system32\drivers\etc\hosts"
             Logfile = "C:\temp\log.txt"
         }
     
         $computers | Invoke-Parallel -parameter $stuff {
             $contentFile = join-path "\\$_\c$" $parameter.contentfile
             Get-Content $contentFile |
                 set-content $parameter.logfile
         }
 
         This example uses the parameter argument.  This parameter is a single object.  To pass multiple items into the script block, we create a custom object (using a PowerShell v3 language) with properties we want to pass in.
 
         Inside the script block, $parameter is used to reference this parameter object.  This example sets a content file, gets content from that file, and sets it to a predefined log file.
 
     .EXAMPLE
         $test = 5
         1..2 | Invoke-Parallel -ImportVariables {$_ * $test}
 
         Add variables from the current session to the session state.  Without -ImportVariables $Test would not be accessible
 
     .EXAMPLE
         $test = 5
         1..2 | Invoke-Parallel -ImportVariables {$_ * $Using:test}
 
         Reference a variable from the current session with the $Using:<Variable> syntax.  Requires PowerShell 3 or later.
 
     .FUNCTIONALITY
         PowerShell Language
 
     .NOTES
         Credit to Boe Prox for the base runspace code and $Using implementation
             http://learn-powershell.net/2012/05/10/speedy-network-information-query-using-powershell/
             https://github.com/proxb/PoshRSJob/
 
         Credit to T Bryce Yehl for the Quiet and NoCloseOnTimeout implementations
 
         Credit to Sergei Vorobev for the many ideas and contributions that have improved functionality, reliability, and ease of use
 
     .LINK
         https://github.com/RamblingCookieMonster/Invoke-Parallel
     #>
     [cmdletbinding(DefaultParameterSetName='ScriptBlock')]
     Param (   
         [Parameter(Mandatory=$false,position=0,ParameterSetName='ScriptBlock')]
             [System.Management.Automation.ScriptBlock]$ScriptBlock,
 
         [Parameter(Mandatory=$false,ParameterSetName='ScriptFile')]
         [ValidateScript({test-path $_ -pathtype leaf})]
             $ScriptFile,
 
         [Parameter(Mandatory=$true,ValueFromPipeline=$true)]
         [Alias('CN','__Server','IPAddress','Server','ComputerName')]    
             [PSObject]$InputObject,
 
             [PSObject]$Parameter,
 
             [switch]$ImportVariables,
 
             [switch]$ImportModules,
 
             [int]$Throttle = 20,
 
             [int]$SleepTimer = 200,
 
             [int]$RunspaceTimeout = 0,
 
 			[switch]$NoCloseOnTimeout = $false,
 
             [int]$MaxQueue,
 
         [validatescript({Test-Path (Split-Path $_ -parent)})]
             [string]$LogFile = "C:\temp\log.log",
 
 			[switch] $Quiet = $false
     )
     
     Begin {
                 
         if( -not $PSBoundParameters.ContainsKey('MaxQueue') )
         {
             if($RunspaceTimeout -ne 0){ $script:MaxQueue = $Throttle }
             else{ $script:MaxQueue = $Throttle * 3 }
         }
         else
         {
             $script:MaxQueue = $MaxQueue
         }
 
         Write-Verbose "Throttle: '$throttle' SleepTimer '$sleepTimer' runSpaceTimeout '$runspaceTimeout' maxQueue '$maxQueue' logFile '$logFile'"
 
         if ($ImportVariables -or $ImportModules)
         {
             $StandardUserEnv = [powershell]::Create().addscript({
 
                 $Modules = Get-Module | Select -ExpandProperty Name
                 $Snapins = Get-PSSnapin | Select -ExpandProperty Name
 
                 $Variables = Get-Variable | Select -ExpandProperty Name
                 
                 @{
                     Variables = $Variables
                     Modules = $Modules
                     Snapins = $Snapins
                 }
             }).invoke()[0]
             
             if ($ImportVariables) {
                 Function _temp {[cmdletbinding()] param() }
                 $VariablesToExclude = @( (Get-Command _temp | Select -ExpandProperty parameters).Keys + $PSBoundParameters.Keys + $StandardUserEnv.Variables )
                 Write-Verbose "Excluding variables $( ($VariablesToExclude | sort ) -join ", ")"
 
                 $UserVariables = @( Get-Variable | Where { -not ($VariablesToExclude -contains $_.Name) } ) 
                 Write-Verbose "Found variables to import: $( ($UserVariables | Select -expandproperty Name | Sort ) -join ", " | Out-String).`n"
 
             }
 
             if ($ImportModules) 
             {
                 $UserModules = @( Get-Module | Where {$StandardUserEnv.Modules -notcontains $_.Name -and (Test-Path $_.Path -ErrorAction SilentlyContinue)} | Select -ExpandProperty Path )
                 $UserSnapins = @( Get-PSSnapin | Select -ExpandProperty Name | Where {$StandardUserEnv.Snapins -notcontains $_ } ) 
             }
         }
 
             
             Function Get-RunspaceData {
                 [cmdletbinding()]
                 param( [switch]$Wait )
 
                 Do {
 
                     $more = $false
 
                     if (-not $Quiet) {
 						Write-Progress  -Activity "Running Query" -Status "Starting threads"`
 							-CurrentOperation "$startedCount threads defined - $totalCount input objects - $script:completedCount input objects processed"`
 							-PercentComplete $( Try { $script:completedCount / $totalCount * 100 } Catch {0} )
 					}
 
                     Foreach($runspace in $runspaces) {
                     
                         $currentdate = Get-Date
                         $runtime = $currentdate - $runspace.startTime
                         $runMin = [math]::Round( $runtime.totalminutes ,2 )
 
                         $log = "" | select Date, Action, Runtime, Status, Details
                         $log.Action = "Removing:'$($runspace.object)'"
                         $log.Date = $currentdate
                         $log.Runtime = "$runMin minutes"
 
                         If ($runspace.Runspace.isCompleted) {
                             
                             $script:completedCount++
                         
                             if($runspace.powershell.Streams.Error.Count -gt 0) {
                                 
                                 $log.status = "CompletedWithErrors"
                                 Write-Verbose ($log | ConvertTo-Csv -Delimiter ";" -NoTypeInformation)[1]
                                 foreach($ErrorRecord in $runspace.powershell.Streams.Error) {
                                     Write-Error -ErrorRecord $ErrorRecord
                                 }
                             }
                             else {
                                 
                                 $log.status = "Completed"
                                 Write-Verbose ($log | ConvertTo-Csv -Delimiter ";" -NoTypeInformation)[1]
                             }
 
                             $runspace.powershell.EndInvoke($runspace.Runspace)
                             $runspace.powershell.dispose()
                             $runspace.Runspace = $null
                             $runspace.powershell = $null
 
                         }
 
                         ElseIf ( $runspaceTimeout -ne 0 -and $runtime.totalseconds -gt $runspaceTimeout) {
                             
                             $script:completedCount++
                             $timedOutTasks = $true
                             
                             $log.status = "TimedOut"
                             Write-Verbose ($log | ConvertTo-Csv -Delimiter ";" -NoTypeInformation)[1]
                             Write-Error "Runspace timed out at $($runtime.totalseconds) seconds for the object:`n$($runspace.object | out-string)"
 
                             if (!$noCloseOnTimeout) { $runspace.powershell.dispose() }
                             $runspace.Runspace = $null
                             $runspace.powershell = $null
                             $completedCount++
 
                         }
                    
                         ElseIf ($runspace.Runspace -ne $null ) {
                             $log = $null
                             $more = $true
                         }
 
                         if($logFile -and $log){
                             ($log | ConvertTo-Csv -Delimiter ";" -NoTypeInformation)[1] | out-file $LogFile -append
                         }
                     }
 
                     $temphash = $runspaces.clone()
                     $temphash | Where { $_.runspace -eq $Null } | ForEach {
                         $Runspaces.remove($_)
                     }
 
                     if($PSBoundParameters['Wait']){ Start-Sleep -milliseconds $SleepTimer }
 
                 } while ($more -and $PSBoundParameters['Wait'])
                 
             }
 
         
 
             if($PSCmdlet.ParameterSetName -eq 'ScriptFile')
             {
                 $ScriptBlock = [scriptblock]::Create( $(Get-Content $ScriptFile | out-string) )
             }
             elseif($PSCmdlet.ParameterSetName -eq 'ScriptBlock')
             {
                 [string[]]$ParamsToAdd = '$_'
                 if( $PSBoundParameters.ContainsKey('Parameter') )
                 {
                     $ParamsToAdd += '$Parameter'
                 }
 
                 $UsingVariableData = $Null
                 
 
                 
                 if($PSVersionTable.PSVersion.Major -gt 2)
                 {
                     $UsingVariables = $ScriptBlock.ast.FindAll({$args[0] -is [System.Management.Automation.Language.UsingExpressionAst]},$True)    
 
                     If ($UsingVariables)
                     {
                         $List = New-Object 'System.Collections.Generic.List`1[System.Management.Automation.Language.VariableExpressionAst]'
                         ForEach ($Ast in $UsingVariables)
                         {
                             [void]$list.Add($Ast.SubExpression)
                         }
 
                         $UsingVar = $UsingVariables | Group SubExpression | ForEach {$_.Group | Select -First 1}
         
                         $UsingVariableData = ForEach ($Var in $UsingVar) {
                             Try
                             {
                                 $Value = Get-Variable -Name $Var.SubExpression.VariablePath.UserPath -ErrorAction Stop
                                 [pscustomobject]@{
                                     Name = $Var.SubExpression.Extent.Text
                                     Value = $Value.Value
                                     NewName = ('$__using_{0}' -f $Var.SubExpression.VariablePath.UserPath)
                                     NewVarName = ('__using_{0}' -f $Var.SubExpression.VariablePath.UserPath)
                                 }
                             }
                             Catch
                             {
                                 Write-Error "$($Var.SubExpression.Extent.Text) is not a valid Using: variable!"
                             }
                         }
                         $ParamsToAdd += $UsingVariableData | Select -ExpandProperty NewName -Unique
 
                         $NewParams = $UsingVariableData.NewName -join ', '
                         $Tuple = [Tuple]::Create($list, $NewParams)
                         $bindingFlags = [Reflection.BindingFlags]"Default,NonPublic,Instance"
                         $GetWithInputHandlingForInvokeCommandImpl = ($ScriptBlock.ast.gettype().GetMethod('GetWithInputHandlingForInvokeCommandImpl',$bindingFlags))
         
                         $StringScriptBlock = $GetWithInputHandlingForInvokeCommandImpl.Invoke($ScriptBlock.ast,@($Tuple))
 
                         $ScriptBlock = [scriptblock]::Create($StringScriptBlock)
 
                         Write-Verbose $StringScriptBlock
                     }
                 }
                 
                 $ScriptBlock = $ExecutionContext.InvokeCommand.NewScriptBlock("param($($ParamsToAdd -Join ", "))`r`n" + $Scriptblock.ToString())
             }
             else
             {
                 Throw "Must provide ScriptBlock or ScriptFile"; Break
             }
 
             Write-Debug "`$ScriptBlock: $($ScriptBlock | Out-String)"
             Write-Verbose "Creating runspace pool and session states"
 
             $sessionstate = [System.Management.Automation.Runspaces.InitialSessionState]::CreateDefault()
             if ($ImportVariables)
             {
                 if($UserVariables.count -gt 0)
                 {
                     foreach($Variable in $UserVariables)
                     {
                         $sessionstate.Variables.Add( (New-Object -TypeName System.Management.Automation.Runspaces.SessionStateVariableEntry -ArgumentList $Variable.Name, $Variable.Value, $null) )
                     }
                 }
             }
             if ($ImportModules)
             {
                 if($UserModules.count -gt 0)
                 {
                     foreach($ModulePath in $UserModules)
                     {
                         $sessionstate.ImportPSModule($ModulePath)
                     }
                 }
                 if($UserSnapins.count -gt 0)
                 {
                     foreach($PSSnapin in $UserSnapins)
                     {
                         [void]$sessionstate.ImportPSSnapIn($PSSnapin, [ref]$null)
                     }
                 }
             }
 
             $runspacepool = [runspacefactory]::CreateRunspacePool(1, $Throttle, $sessionstate, $Host)
             $runspacepool.Open() 
 
             Write-Verbose "Creating empty collection to hold runspace jobs"
             $Script:runspaces = New-Object System.Collections.ArrayList        
         
             $bound = $PSBoundParameters.keys -contains "InputObject"
             if(-not $bound)
             {
                 [System.Collections.ArrayList]$allObjects = @()
             }
 
             if( $LogFile ){
                 New-Item -ItemType file -path $logFile -force | Out-Null
                 ("" | Select Date, Action, Runtime, Status, Details | ConvertTo-Csv -NoTypeInformation -Delimiter ";")[0] | Out-File $LogFile
             }
 
             $log = "" | Select Date, Action, Runtime, Status, Details
                 $log.Date = Get-Date
                 $log.Action = "Batch processing started"
                 $log.Runtime = $null
                 $log.Status = "Started"
                 $log.Details = $null
                 if($logFile) {
                     ($log | convertto-csv -Delimiter ";" -NoTypeInformation)[1] | Out-File $LogFile -Append
                 }
 
 			$timedOutTasks = $false
 
     }
 
     Process {
 
         if($bound)
         {
             $allObjects = $InputObject
         }
         Else
         {
             [void]$allObjects.add( $InputObject )
         }
     }
 
     End {
         
         Try
         {
             $totalCount = $allObjects.count
             $script:completedCount = 0
             $startedCount = 0
 
             foreach($object in $allObjects){
         
                     
                     $powershell = [powershell]::Create()
                     
                     if ($VerbosePreference -eq 'Continue')
                     {
                         [void]$PowerShell.AddScript({$VerbosePreference = 'Continue'})
                     }
 
                     [void]$PowerShell.AddScript($ScriptBlock).AddArgument($object)
 
                     if ($parameter)
                     {
                         [void]$PowerShell.AddArgument($parameter)
                     }
 
                     if ($UsingVariableData)
                     {
                         Foreach($UsingVariable in $UsingVariableData) {
                             Write-Verbose "Adding $($UsingVariable.Name) with value: $($UsingVariable.Value)"
                             [void]$PowerShell.AddArgument($UsingVariable.Value)
                         }
                     }
 
                     $powershell.RunspacePool = $runspacepool
     
                     $temp = "" | Select-Object PowerShell, StartTime, object, Runspace
                     $temp.PowerShell = $powershell
                     $temp.StartTime = Get-Date
                     $temp.object = $object
     
                     $temp.Runspace = $powershell.BeginInvoke()
                     $startedCount++
 
                     Write-Verbose ( "Adding {0} to collection at {1}" -f $temp.object, $temp.starttime.tostring() )
                     $runspaces.Add($temp) | Out-Null
             
                     Get-RunspaceData
 
                     $firstRun = $true
                     while ($runspaces.count -ge $Script:MaxQueue) {
 
                         if($firstRun){
                             Write-Verbose "$($runspaces.count) items running - exceeded $Script:MaxQueue limit."
                         }
                         $firstRun = $false
                     
                         Get-RunspaceData
                         Start-Sleep -Milliseconds $sleepTimer
                     
                     }
 
             }
                      
             Write-Verbose ( "Finish processing the remaining runspace jobs: {0}" -f ( @($runspaces | Where {$_.Runspace -ne $Null}).Count) )
             Get-RunspaceData -wait
 
             if (-not $quiet) {
 			    Write-Progress -Activity "Running Query" -Status "Starting threads" -Completed
 		    }
         }
         Finally
         {
             if ( ($timedOutTasks -eq $false) -or ( ($timedOutTasks -eq $true) -and ($noCloseOnTimeout -eq $false) ) ) {
 	            Write-Verbose "Closing the runspace pool"
 			    $runspacepool.close()
             }
 
             [gc]::Collect()
         }
     }
 }
 
 
 if($PSVersionTable.PSVersion.Major -lt 3){$WorkingDirectory = split-path -parent $MyInvocation.MyCommand.Definition}
 else{$WorkingDirectory = $PSScriptRoot}
 
 $Masterlist = gc $WorkingDirectory\testing.txt
 
 Invoke-Parallel -throttle 20 -RunspaceTimeout 60 -inputobject $Masterlist -NoCloseOnTimeout -importvariables -importmodules -scriptblock {
 	Set-Alias psexec "$WorkingDirectory\psexec\psexec.exe"
 	if (test-connection -count 1 -quiet -computername $_ -buffersize 16) {
 
 		Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "yellow" $_ "Running WMI Consistency Check winmgmt /verifyrepository"
 		$computername = "\\$_"
 		$Scriptblock = "winmgmt /verifyrepository"
 
 		$commandLine = "echo . | powershell -Output XML "
 
 		$commandBytes = [System.Text.Encoding]::Unicode.GetBytes($scriptblock)
 		$encodedCommand = [Convert]::ToBase64String($commandBytes)
 		$commandLine += "-EncodedCommand $encodedCommand"
 
 		$errorOutput = [IO.Path]::GetTempFileName()
 		$output = psexec /acceptEula $computername cmd /c $commandLine 2>$errorOutput
 
 		$errorContent = Get-Content $errorOutput
 		Remove-Item $errorOutput
 		if($errorContent -match "(Access is denied)|(failure)|(Couldn't)") {
 			$OFS = "`n"
 			$errorMessage = "Could not execute remote expression. "
 			$errorMessage += "Ensure that your account has administrative " +
 				"privileges on the target machine.`n"
 			$errorMessage += ($errorContent -match "psexec.exe :")
 
 			Write-Error $errorMessage
 		}
 
 		if ($output -eq "WMI repository is consistent") {
 			Write-Host $(Get-Date -format "dd_MM_yyyy_HH:mm:ss") -foregroundcolor "green" $_ "WMI Check: Success"
 			$wmicheckresults = [pscustomobject] @{
 			Computer="$_";
 			WMIConsistency="$True"}
 		}
 		else {
 			$wmicheckresults = [pscustomobject] @{
 			Computer="$_";
 			WMIConsistency="$False"}
 		}
 		$wmicheckresults | Export-Clixml -path $WorkingDirectory\(($wmicheckresults).computer)_tempfile.txt
 	}
 }
 
 Start-Sleep -seconds 1
 $Results = gci $WorkingDirectory -filter "*tempfile.txt" | % {import-clixml $_.fullname}
 $Results
`

