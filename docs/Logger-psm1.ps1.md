---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.4
Encoding: ascii
License: cc0
PoshCode ID: 4958
Published Date: 2017-03-05t01
Archived Date: 2017-02-26t05
---

# logger.psm1 - 

## Description

the simplest logger. in your script just import-module logger and debug, verbose, warnings and errors are logged to file.

## Comments



## Usage



## TODO



## module

`get-relativepath`

## Code

`#
 #
 <#
    Name     : Universal Log4Net Logging Module (Logger.psm1)
    Version  : 0.4
    Author   : Joel Bennett (MVP)
    Site     : http://www.HuddledMasses.org/
 
    Version History:
    0.4 - Bugfix, Viewer and Documentation release.
          Fixed a few typo-bugs
          Added documentation (man page) comments for Get-Logger.
          Changed a few parameter names (sorry) to make the default parameters more unique (so you have to type less on the command-line)
          Changed the default logger to use the logger module's folder as a backup
             (Some people might not have the Profile path -- this module could be installed anywhere and loaded by file name)
          Fixed up the xml output with a nice stylesheet http`://poshcode.org/1750 that formats and makes the page refresh.
    
    0.3 - Cleanupable release.
          Added Udp, Email,  Xml and RollingXml, as well as a "Chainsaw":http`://logging.apache.org/log4j/docs/chainsaw.html logger based on "Szymon Kobalczyk's configuration":http`://geekswithblogs.net/kobush/archive/2005/07/15/46627.aspx.
          Note: there is a "KNOWN BUG with Log4Net UDP":http`://odalet.wordpress.com/2007/01/13/log4net-ip-parsing-bug-when-used-with-framework-net-20/ which can be patched, but hasn't been released.
          Added a Close-Logger method to clean up the Xml logger 
          NOTE: the Rolling XML Logger produces invalid XML, and the XML logger only produces valid xml after it's been closed...
                I did contribute an "XSLT stylesheet for Log4Net":http`://poshcode.org/1746 which you could use as well
          
    0.2 - Configless release. 
          Now configures with inline XML, and supports switches to create "reasonable" default loggers
          Changed all the functions to Write-*Log so they don't overwrite the cmdlets
          Added -Logger parameter to take the name of the logger to use (it must be created beforehand via Get-Logger)
          Created aliases for Write-* to override the cmdlets -- these are easy for users to remove without breaking the module
          ** NEED to write some docs, but basically, this is stupid simple to use, just:
             Import-Module Logger
             Write-Verbose "This message will be saved in your profile folder in a file named PowerShellLogger.log (by default)"
          To change the defaults for your system, edit the last line in the module!!
    0.1 - Initial release. http`://poshcode.org/1744 (Required config: http`://poshcode.org/1743)
 
    Uses Log4Net : http`://logging.apache.org/log4net/download.html
    Documentation: http`://logging.apache.org/log4net/release/sdk/
    
    NOTES:
    By default, this overwrites the Write-* cmdlets for Error, Warning, Debug, Verbose, and even Host.
    This means that you may end up logging a lot of stuff you didn't intend to log (ie: verbose output from other scripts)
    
    To avoid this behavior, remove the aliases after importing it
    Import-Module Logger; Remove-Item Alias:Write-*
    Write-Warning "This is your warning"
    Write-Debug   "You should know that..."
    Write-Verbose "Everything would be logged, otherwise"
 
    ***** NOTE: IT ONLY OVERRIDES THE DEFAULTS FOR SCRIPTS *****
    It currently has no effect on error/verbose/warning that is logged from cmdlets.
    
 #>
 
 Add-Type -Path $PSScriptRoot\log4net.dll
 
 function Get-RelativePath {
 <#
 .SYNOPSIS
    Get a path to a file (or folder) relative to another folder
 .DESCRIPTION
    Converts the FilePath to a relative path rooted in the specified Folder
 .PARAMETER Folder
    The folder to build a relative path from
 .PARAMETER FilePath
    The File (or folder) to build a relative path TO
 .PARAMETER Resolve
    If true, the file and folder paths must exist
 .Example
    Get-RelativePath ~\Documents\WindowsPowerShell\Logs\ ~\Documents\WindowsPowershell\Modules\Logger\log4net.xslt
    
    ..\Modules\Logger\log4net.xslt
    
    Returns a path to log4net.xslt relative to the Logs folder
 #>
 [CmdletBinding()]
 param(
    [Parameter(Mandatory=$true, Position=0)]
    [string]$Folder
 , 
    [Parameter(Mandatory=$true, Position=1, ValueFromPipelineByPropertyName=$true)]
    [Alias("FullName")]
    [string]$FilePath
 ,
    [switch]$Resolve
 )
 process {
    $from = $Folder = split-path $Folder -NoQualifier -Resolve:$Resolve
    $to = $filePath = split-path $filePath -NoQualifier -Resolve:$Resolve
 
    while($from -and $to -and ($from -ne $to)) {
       if($from.Length -gt $to.Length) {
          $from = split-path $from
       } else {
          $to = split-path $to
       }
    }
 
    $filepath = $filepath -replace "^"+[regex]::Escape($to)+"\\"
    $from = $Folder
    while($from -and $to -and $from -gt $to ) {
       $from = split-path $from
       $filepath = join-path ".." $filepath
    }
    Write-Output $filepath
 }
 }
 
 function Get-Logger {
 <#
 .SYNOPSIS
    Get an existing Logger by name, or create a new logger
 .DESCRIPTION
    Returns a logger matching the name (wildcards allowed) provided. 
    
    If the logger already exists, it is returned with it's settings as-is, unless the -Force switch is specified, in which case the new settings are used
    
    If only one logger matches the name, that logger becomes the new default logger.
 
 .PARAMETER Name
    The name of the logger to find or create. If no name is specified, all existing loggers are returned.
 
 .PARAMETER Level
    The level at which to log in this new logger. Defaults to "DEBUG" 
    Possible levels are as follows (each level includes all levels above it):
    
    FATAL
    ERROR
    WARN  (aka WARNING)
    INFO  (aka VERBOSE, HOST)
    DEBUG
    
 .PARAMETER MessagePattern
    The pattern for loggers which use patterns (mostly the file loggers). Defaults to: 
    "%date %-5level %logger [%property{NDC}] - %message%newline"
    
    For a complete list of possible pattern names, see:
    http://logging.apache.org/log4net/release/sdk/log4net.Layout.PatternLayout.html
    
 .PARAMETER Folder
    The folder where log files are kept. Defaults to your Documents\WindowsPowerShell folder.
    NOTE: if the specified folder does not exist, the fallback is your Documents\WindowsPowerShell folder,
          but if that doesn't exist, the folder where this file is stored is used.
 
 .PARAMETER EmailTo
    An email address to send WARNING or above messages to. REQUIRES that your $PSEmailServer variable be set
 .PARAMETER Console
    Creates a colored console logger
 .PARAMETER EventLog
    Creates an EventLog logger
 .PARAMETER TraceLog
    Creates a .Net Trace logger
 .PARAMETER DebugLog
    Creates a .Net Debug logger
 .PARAMETER FileLog
    Creates a file logger. Note the LogLock parameter!
 .PARAMETER RollingFileLog
    Creates a rolling file logger with a max size of 250KB. Note the LogLock parameter!   
 .PARAMETER XmlLog
    Creates an Xml-formatted file logger. Note the LogLock parameter!
    
    Note: the XmlLog will output an XML Header and will add a footer when the logger is closed.
    This results in a log file which is readable in xml viewers like IE, particularly if you have a copy of the XSLT stylesheet for Log4Net (http://poshcode.org/1750) named log4net.xslt in the same folder with the log file.
    
    WARNING: Because of this, it does not APPEND to the file, but overwrites it each time you create the logger.
 .PARAMETER RollingXmlLog
    Creates a rolling file Xml logger with a max size of 500KB. Note the LogLock parameter!
    
    Note: the rolling xml logger cannot create "valid" xml because it appends to the end of the file (and therefore can't guarantee the file is well-formed XML).
    In order to get a valid Xml file, you can use a "*View.xml" file with contents like this (which this script will create):
    
    <?xml version="1.0" ?>
    <?xml-stylesheet type="text/xsl" href="log4net.xslt"?>
    <!DOCTYPE events [<!ENTITY data SYSTEM "PowerShellLogger.xml">]>
    <log4net:events version="1.2" xmlns:log4net="http://logging.apache.org/log4net/schemas/log4net-events-1.2">
       &data;
    </log4net:events>
 .PARAMETER LogLock
    Determines the type of file locking used (defaults to SHARED): 
    SHARED is the "MinimalLocking" model which opens the file once for each AcquireLock/ReleaseLock cycle, thus holding the lock for the minimal amount of time. This method of locking is considerably slower than...
    
    EXCLUSIVE is the "ExclusiveLocking" model which opens the file once for writing and holds it open until the logger is closed, maintaining an exclusive lock on the file the whole time..
 .PARAMETER UdpSend
    Creates an UdpAppender which sends messages to the localmachine port 8080
    We'll probably tweak this in a future release, but for now if you need to change that you need to edit the script
 .PARAMETER ChainsawSend
    Like UdpSend, creates an UdpAppender which sends messages to the localmachine port 8080. 
    This logger uses the log4j xml formatter which is somewhat different than the default, and uses their namespace.   
 .PARAMETER Force
    Force resetting the logger switches even if the logger already exists
 #>
 param( 
    [Parameter(Mandatory=$false, Position=0)]
    [string]$Name = "*"
 ,
    [Parameter(Mandatory=$false)]
    [ValidateSet("DEBUG","INFO","WARN","ERROR","FATAL","VERBOSE","HOST","WARNING")]
    [string]$Level = "DEBUG"
 ,
    [Parameter(Mandatory=$false)]
    $MessagePattern = "%date %-5level %logger [%property{NDC}] - %message%newline"
 ,
    [Parameter(Mandatory=$false)]
    [string]$Folder = $(Split-Path $Profile.CurrentUserAllHosts)
    
 ,  [String]$EmailTo
 ,  [Switch]$Force
 ,  [Switch]$ConsoleLog
 ,  [Switch]$EventLog
 ,  [Switch]$TraceLog
 ,  [Switch]$DebugLog
 ,  [Switch]$FileLog
 ,  [Switch]$RollingFileLog
 ,  [Switch]$XmlLog
 ,  [Switch]$RollingXmlLog
 ,  [Switch]$UdpSend
 ,  [Switch]$ChainsawSend
 ,
    [Parameter(Mandatory=$false, Position=99)]
    [ValidateSet("Shared","Exclusive")]
    [String]$LogLock = "Shared"
 )
    if(!(Test-Path $Folder)) {
       $Folder = Split-Path $Profile.CurrentUserAllHosts
       if(!(Test-Path $Folder)) {
          $Folder = $PSScriptRoot
       }
    }
    
    
    Remove-Variable Loggers -EA 0
    [log4net.LogManager]::GetCurrentLoggers() | Where-Object { $_.Logger.Name -Like $Name } | Tee-Object -Variable Loggers
    if((!$Loggers -or $Force) -and !$Name.Contains('*')) {
       if($Level -eq "VERBOSE") { $Level = "INFO" }
       if($Level -eq "HOST")    { $Level = "INFO" }
       if($Level -eq "WARNING") { $Level = "WARN" }
 
       $AppenderRefs = ''
       if($Email)        { 
          if(!$PSEmailServer) { throw "You need to specify your SMTP mail server by setting the global $PSEmailServer variable" }
          $AppenderRefs += "<appender-ref ref=""SmtpAppender"" />`n"
          $Null,$Domain = $email -split "@",2
       } 
       
       if($LogLock -eq "Shared") { 
          $LockMode = "MinimalLock"
       } else {
          $LockMode = "ExclusiveLock"
       }
       $xslt = ""
       if(Test-Path $PSScriptRoot\log4net.xslt) {
          $xslt = @"
 <?xml-stylesheet type="text/xsl" href="$(Get-RelativePath $Folder $PSScriptRoot\log4net.xslt)"?>
 "@
       }
 
 
       if($EventLog)        { $AppenderRefs += "<appender-ref ref=""EventLogAppender"" />`n" }
       if($TraceLog)        { $AppenderRefs += "<appender-ref ref=""TraceAppender"" />`n" }
       if($DebugLog){ $AppenderRefs += "<appender-ref ref=""DebugAppender"" />`n" }
       if($FileLog)         { $AppenderRefs += "<appender-ref ref=""FileAppender"" />`n" }
       if($RollingFileLog)  { $AppenderRefs += "<appender-ref ref=""RollingFileAppender"" />`n" }
       if($UdpSend)         { $AppenderRefs += "<appender-ref ref=""UdpAppender"" />`n" } 
       if($ChainsawSend)    { $AppenderRefs += "<appender-ref ref=""UdpLog4JAppender"" />`n" } 
       if($XmlLog)          { $AppenderRefs += "<appender-ref ref=""XmlAppender"" />`n" } 
       if($RollingXmlLog)   { $AppenderRefs += "<appender-ref ref=""RollingXmlAppender"" />`n"
          if($VerbosePreference -gt "SilentlyContinue") { "Created XML viewer for you at: ${Folder}\${Name}View.Xml" }
          Set-Content "${Folder}\${Name}View.Xml" @"
 <?xml version="1.0" ?>
 $xslt
 <!DOCTYPE events [<!ENTITY data SYSTEM "$Name.xml">]>
 <events version="1.2" xmlns:log4net="http`://logging.apache.org/log4net/schemas/log4net-events-1.2">
 <logname>$Name</logname>
    &data;
 </events>
 "@      
       }
       $xslt = $xslt -replace "<","&lt;" -replace ">","&gt;" -replace '"',"'"
       
       if($ConsoleLog -or ($AppenderRefs.Length -eq 0)) { $AppenderRefs += "<appender-ref ref=""ColoredConsoleAppender"" />`n" }
 
       [log4net.LogManager]::GetLogger($Name) | Tee-Object -Variable Script:Logger | Where { $Loggers -notcontains $_ }
      
       [xml]$config = @"
 <log4net>
    <appender name="ColoredConsoleAppender" type="log4net.Appender.ColoredConsoleAppender">
       <mapping>
          <level value="FATAL" />
          <foreColor value="Red, HighIntensity" />
          <backColor value="White, HighIntensity" />
       </mapping>
       <mapping>
          <level value="ERROR" />
          <foreColor value="Red, HighIntensity" />
       </mapping>
       <mapping>
          <level value="DEBUG" />
          <foreColor value="Green, HighIntensity" />
       </mapping>
       <mapping>
          <level value="INFO" />
          <foreColor value="Cyan, HighIntensity" />
       </mapping>
       <mapping>
          <level value="WARN" />
          <foreColor value="Yellow, HighIntensity" />
       </mapping>
       <layout type="log4net.Layout.PatternLayout">
          <conversionPattern value="$MessagePattern" />
       </layout>
    </appender>
    <appender name="EventLogAppender" type="log4net.Appender.EventLogAppender" >
       <applicationName value="$Name" />
       <layout type="log4net.Layout.PatternLayout">
          <conversionPattern value="$MessagePattern" />
       </layout>
    </appender>
    <appender name="TraceAppender" type="log4net.Appender.TraceAppender">
       <layout type="log4net.Layout.PatternLayout">
          <conversionPattern value="$MessagePattern" />
       </layout>
    </appender>
    <appender name="DebugAppender" type="log4net.Appender.DebugAppender">
       <layout type="log4net.Layout.PatternLayout">
          <conversionPattern value="$MessagePattern" />
       </layout>
    </appender>
    <appender name="FileAppender" type="log4net.Appender.FileAppender">
       <file value="$Folder\$Name.Log" />
       <appendToFile value="true" />
       <lockingModel type="log4net.Appender.FileAppender+$LockMode" />
       <layout type="log4net.Layout.PatternLayout">
          <conversionPattern value="$MessagePattern" />
       </layout>
    </appender>
    <appender name="XmlAppender" type="log4net.Appender.FileAppender">
       <file value="$Folder\$Name.xml" />
       <appendToFile value="false" />
       <lockingModel type="log4net.Appender.FileAppender+$LockMode" />
       <layout type="log4net.Layout.XmlLayout">
          <Header value="&lt;?xml version='1.0' ?&gt;
 $xslt
 &lt;events version='1.2' xmlns='http`://logging.apache.org/log4net/schemas/log4net-events-1.2'&gt;
 "/>
          <Footer value="&lt;/events&gt;"/>
       </layout>
    </appender>
    <appender name="RollingFileAppender" type="log4net.Appender.RollingFileAppender">
       <file value="$Folder\$Name.Log" />
       <appendToFile value="true" />
       <maximumFileSize value="250KB" />
       <maxSizeRollBackups value="2" />
       <lockingModel type="log4net.Appender.FileAppender+$LockMode" />
       <layout type="log4net.Layout.PatternLayout">
          <conversionPattern value="$MessagePattern" />
       </layout>
    </appender>
    <appender name="RollingXmlAppender" type="log4net.Appender.RollingFileAppender">
       <file value="$Folder\$Name.xml" />
       <appendToFile value="true" />
       <maximumFileSize value="500KB" />
       <maxSizeRollBackups value="2" />
       <lockingModel type="log4net.Appender.FileAppender+$LockMode" />
       <layout type="log4net.Layout.XmlLayout">
          <prefix value="" />
       </layout>
    </appender>
    <appender name="UdpAppender" type="log4net.Appender.UdpAppender">
       <RemoteAddress value="localhost" />
       <RemotePort value="8080" />
       <Encoding value="utf-8" />
       <layout type="log4net.Layout.XmlLayout">
          <Prefix value="" />
       </layout>
       <threshold value="DEBUG" />
    </appender>
    <appender name="UdpLog4JAppender" type="log4net.Appender.UdpAppender">
       <RemoteAddress value="127.0.0.1" />
       <RemotePort value="8080" />
       <Encoding value="utf-8" />
       <layout type="log4net.Layout.XmlLayoutSchemaLog4j, log4net" />
       <threshold value="DEBUG" />
    </appender>
    <appender name="SmtpAppender" type="log4net.Appender.SmtpAppender">
       <to value="$EmailTo" />
       <from value="PoshLogger@$Domain" />
       <subject value="PowerShell Logger Message" />
       <smtpHost value="$PSEmailServer" />
       <bufferSize value="512" />
       <lossy value="true" />
       <evaluator type="log4net.Core.LevelEvaluator">
          <threshold value="WARN"/>
       </evaluator>
       <layout type="log4net.Layout.PatternLayout">
          <conversionPattern value="%newline$MessagePattern%newline%newline" />
       </layout>
    </appender>
    <root>
       <level value="DEBUG" />
    </root>
    <logger name="$Name">
       <level value="$Level" />
       $AppenderRefs
    </logger>
 </log4net>
 "@
       [log4net.Config.XmlConfigurator]::Configure( $config.log4net )
    } 
    elseif($Loggers -and @($Loggers).Count -eq 1) {
       $script:Logger = @($Loggers)[0]
    }
 }
 
 function Set-Logger {
 param(
    [Parameter(Mandatory=$true, ValueFromPipeline=$true)]
    [log4net.Core.LogImpl]$Logger
 )
    $script:Logger = $Logger
 }
 
 function Close-Logger {
 param(
    [Parameter(Mandatory=$false, ValueFromPipeline=$true)]
    [log4net.Core.LogImpl]$Logger
 )
 PROCESS {
    if($Logger) {
       $Logger.Logger.CloseNestedAppenders()
       $Logger.Logger.RemoveAllAppenders()
    } else {
       [log4net.LogManager]::Shutdown()
    }
 }
 }
 
 
 
 function Push-LogContext {
 param( 
    [Parameter(Mandatory=$true)]
    [string]$Name
 )
    [log4net.NDC]::Push($Name)
 }
 function Pop-LogContext {
    [log4net.NDC]::Pop()
 }
 
 function Write-DebugLog {
 [CmdletBinding()]
 param(
     [Parameter(Mandatory=$true, Position=0, ValueFromPipeline=$true)]
     [Alias('Msg')]
     [AllowEmptyString()]
     [System.String]
     ${Message}
 ,
    [Parameter(Mandatory=$false, Position=99)]
    ${Logger})
 
 begin
 {
     try {
         if($PSBoundParameters.ContainsKey("Logger")) {
             if($Logger -is [log4net.Core.LogImpl]) { Set-Logger $Logger } else { $script:Logger = Get-Logger "$Logger" }
             $null = $PSBoundParameters.Remove("Logger")
         }
         
         $outBuffer = $null
         if ($PSBoundParameters.TryGetValue('OutBuffer', [ref]$outBuffer))
         {
             $PSBoundParameters['OutBuffer'] = 1
         }
         $wrappedCmd = $ExecutionContext.InvokeCommand.GetCommand('Write-Debug', [System.Management.Automation.CommandTypes]::Cmdlet)
         $scriptCmd = {& $wrappedCmd @PSBoundParameters }
         $steppablePipeline = $scriptCmd.GetSteppablePipeline($myInvocation.CommandOrigin)
         $steppablePipeline.Begin($PSCmdlet)
     } catch {
         throw
     }
 }
 
 process
 {
     try {
         $steppablePipeline.Process($_)
     } catch {
         throw
     }
 }
 
 end
 {
     try {
         $steppablePipeline.End()
     } catch {
         throw
     }
 }
 <#
 
 .ForwardHelpTargetName Write-Debug
 .ForwardHelpCategory Cmdlet
 
 #>
 }
 function Write-VerboseLog {
 
 [CmdletBinding()]
 param(
     [Parameter(Mandatory=$true, Position=0, ValueFromPipeline=$true)]
     [Alias('Msg')]
     [AllowEmptyString()]
     [System.String]
     ${Message}
 ,
    [Parameter(Mandatory=$false, Position=99)]
    ${Logger})
 
 begin
 {
     try {
         if($PSBoundParameters.ContainsKey("Logger")) {
             if($Logger -is [log4net.Core.LogImpl]) { Set-Logger $Logger } else { $script:Logger = Get-Logger "$Logger" }
             $null = $PSBoundParameters.Remove("Logger")
         }
 
         $outBuffer = $null
         if ($PSBoundParameters.TryGetValue('OutBuffer', [ref]$outBuffer))
         {
             $PSBoundParameters['OutBuffer'] = 1
         }
         $wrappedCmd = $ExecutionContext.InvokeCommand.GetCommand('Write-Verbose', [System.Management.Automation.CommandTypes]::Cmdlet)
         $scriptCmd = {& $wrappedCmd @PSBoundParameters }
         $steppablePipeline = $scriptCmd.GetSteppablePipeline($myInvocation.CommandOrigin)
         $steppablePipeline.Begin($PSCmdlet)
     } catch {
         throw
     }
 }
 
 process
 {
     try {
         $script:logger.info($Message)
         $steppablePipeline.Process($_)
     } catch {
         throw
     }
 }
 
 end
 {
     try {
         $steppablePipeline.End()
     } catch {
         throw
     }
 }
 <#
 
 .ForwardHelpTargetName Write-Verbose
 .ForwardHelpCategory Cmdlet
 
 #>
 }
 function Write-WarningLog {
 [CmdletBinding()]
 param(
     [Parameter(Mandatory=$true, Position=0, ValueFromPipeline=$true)]
     [Alias('Msg')]
     [AllowEmptyString()]
     [System.String]
     ${Message}
 ,
    [Parameter(Mandatory=$false, Position=99)]
    ${Logger})
 
 begin
 {
     try {
         if($PSBoundParameters.ContainsKey("Logger")) {
             if($Logger -is [log4net.Core.LogImpl]) { Set-Logger $Logger } else { $script:Logger = Get-Logger "$Logger" }
             $null = $PSBoundParameters.Remove("Logger")
         }
 
         $outBuffer = $null
         if ($PSBoundParameters.TryGetValue('OutBuffer', [ref]$outBuffer))
         {
             $PSBoundParameters['OutBuffer'] = 1
         }
         $wrappedCmd = $ExecutionContext.InvokeCommand.GetCommand('Write-Warning', [System.Management.Automation.CommandTypes]::Cmdlet)
         $scriptCmd = {& $wrappedCmd @PSBoundParameters }
         $steppablePipeline = $scriptCmd.GetSteppablePipeline($myInvocation.CommandOrigin)
         $steppablePipeline.Begin($PSCmdlet)
     } catch {
         throw
     }
 }
 
 process
 {
     try {
         $steppablePipeline.Process($_)
     } catch {
         throw
     }
 }
 
 end
 {
     try {
         $steppablePipeline.End()
     } catch {
         throw
     }
 }
 <#
 
 .ForwardHelpTargetName Write-Warning
 .ForwardHelpCategory Cmdlet
 
 #>
 }
 function Write-ErrorLog {
 [CmdletBinding(DefaultParameterSetName='NoException')]
 param(
     [Parameter(ParameterSetName='WithException', Mandatory=$true)]
     [System.Exception]
     ${Exception},
 
     [Parameter(ParameterSetName='NoException', Mandatory=$true, Position=0, ValueFromPipeline=$true)]
     [Parameter(ParameterSetName='WithException')]
     [Alias('Msg')]
     [AllowNull()]
     [AllowEmptyString()]
     [System.String]
     ${Message},
 
     [Parameter(ParameterSetName='ErrorRecord', Mandatory=$true)]
     [System.Management.Automation.ErrorRecord]
     ${ErrorRecord},
 
     [Parameter(ParameterSetName='NoException')]
     [Parameter(ParameterSetName='WithException')]
     [System.Management.Automation.ErrorCategory]
     ${Category},
 
     [Parameter(ParameterSetName='WithException')]
     [Parameter(ParameterSetName='NoException')]
     [System.String]
     ${ErrorId},
 
     [Parameter(ParameterSetName='NoException')]
     [Parameter(ParameterSetName='WithException')]
     [System.Object]
     ${TargetObject},
 
     [System.String]
     ${RecommendedAction},
 
     [Alias('Activity')]
     [System.String]
     ${CategoryActivity},
 
     [Alias('Reason')]
     [System.String]
     ${CategoryReason},
 
     [Alias('TargetName')]
     [System.String]
     ${CategoryTargetName},
 
     [Alias('TargetType')]
     [System.String]
     ${CategoryTargetType}
 ,
    [Parameter(Mandatory=$false, Position=99)]
    ${Logger})
 
 begin
 {
     try {
         if($PSBoundParameters.ContainsKey("Logger")) {
             if($Logger -is [log4net.Core.LogImpl]) { Set-Logger $Logger } else { $script:Logger = Get-Logger "$Logger" }
             $null = $PSBoundParameters.Remove("Logger")
         }
 
         $outBuffer = $null
         if ($PSBoundParameters.TryGetValue('OutBuffer', [ref]$outBuffer))
         {
             $PSBoundParameters['OutBuffer'] = 1
         }
         $wrappedCmd = $ExecutionContext.InvokeCommand.GetCommand('Write-Error', [System.Management.Automation.CommandTypes]::Cmdlet)
         $scriptCmd = {& $wrappedCmd @PSBoundParameters }
         $steppablePipeline = $scriptCmd.GetSteppablePipeline($myInvocation.CommandOrigin)
         $steppablePipeline.Begin($PSCmdlet)
     } catch {
         throw
     }
 }
 
 process
 {
     try {
         $steppablePipeline.Process($_)
     } catch {
         throw
     }
 }
 
 end
 {
     try {
         $steppablePipeline.End()
     } catch {
         throw
     }
 }
 <#
 
 .ForwardHelpTargetName Write-Error
 .ForwardHelpCategory Cmdlet
 
 #>
 }
 function Write-HostLog {
 [CmdletBinding()]
 param(
     [Parameter(Position=0, ValueFromPipeline=$true, ValueFromRemainingArguments=$true)]
     [System.Object]
     ${Object},
 
     [Switch]
     ${NoNewline},
 
     [System.Object]
     ${Separator} = $OFS,
 
     [System.ConsoleColor]
     ${ForegroundColor},
 
     [System.ConsoleColor]
     ${BackgroundColor}
 ,
    [Parameter(Mandatory=$false, Position=99)]
    ${Logger})
 
 begin
 {
     try {
         if($PSBoundParameters.ContainsKey("Logger")) {
             if($Logger -is [log4net.Core.LogImpl]) { Set-Logger $Logger } else { $script:Logger = Get-Logger "$Logger" }
             $null = $PSBoundParameters.Remove("Logger")
         }
 
         $outBuffer = $null
         if ($PSBoundParameters.TryGetValue('OutBuffer', [ref]$outBuffer))
         {
             $PSBoundParameters['OutBuffer'] = 1
         }
         $wrappedCmd = $ExecutionContext.InvokeCommand.GetCommand('Write-Host', [System.Management.Automation.CommandTypes]::Cmdlet)
         $scriptCmd = {& $wrappedCmd @PSBoundParameters }
         $steppablePipeline = $scriptCmd.GetSteppablePipeline($myInvocation.CommandOrigin)
         $steppablePipeline.Begin($PSCmdlet)
     } catch {
         throw
     }
 }
 
 process
 {
     try {
         $steppablePipeline.Process($_)
     } catch {
         throw
     }
 }
 
 end
 {
     try {
         $steppablePipeline.End()
     } catch {
         throw
     }
 }
 <#
 
 .ForwardHelpTargetName Write-Host
 .ForwardHelpCategory Cmdlet
 
 #>
 }
 
 Set-Alias Write-Debug Write-DebugLog
 Set-Alias Write-Verbose Write-VerboseLog
 Set-Alias Write-Warning Write-WarningLog
 Set-Alias Write-Error Write-ErrorLog
 
 Export-ModuleMember -Function * -Alias *
 
 $script:Logger = Get-Logger "PowerShellLogger" -RollingFile
 
`

