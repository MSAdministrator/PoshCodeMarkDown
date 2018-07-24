---
Author: glnsize
Publisher: 
Copyright: 
Email: 
Version: 9.0
Encoding: ascii
License: cc0
PoshCode ID: 1065
Published Date: 
Archived Date: 2009-05-04t20
---

# build-tfsprojects - 

## Description

moved source from poshcomm http

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #-------------------------------------------------------------------------    
 #-------------------------------------------------------------------------   
 Param(
     [switch]
     $CSRWEB,
     [switch]
     $CSRWS,
     [switch]
     $CSRServices,
     [string]
     $LogPath,
     [String]
     $Outdir = "C:\Application\CSR"
 )
 Begin 
 {
 
     $MSBUILD="c:\WINDOWS\Microsoft.NET\Framework\v3.5\MsBuild"
     $WEBPROJECTOUTPUTDIR="$OUTDIR\CSRWeb"
     $script:Log=@()
     
     #-------------------------------------------------------------------------    
     Function SetTFS
     {
         $SCRIPT:tfsServer = "172.29.4.179"
         $script:userName = [system.environment]::UserName; 
         $script:computerName = [system.environment]::machinename
         $script:CSRDIR = "C:\Brassring2\Application\"; 
         $script:WEBPROJECTOUTPUTDIR="$Outdir\CSRWeb"
         $script:WEBPROJECTOUTPUTDIR1="$Outdir\CSRWS"
         $script:WEBPROJECTOUTPUTDIR2="$Outdir\CSRServices"
         $script:REFPATH="referenceToAllAssembiesUsedInTheProjectSeperatedbySemiColon"
         $script:MSBUILD="c:\WINDOWS\Microsoft.NET\Framework\v3.5\MsBuild"
         $script:failed = $false
 
 
 
         $key = Get-ItemProperty HKLM:\SOFTWARE\Microsoft\VisualStudio\9.0
         $dir = [string] (Get-ItemProperty $key.InstallDir)
         $script:tf = "$dir\tf.exe"
 
     #-------------------------------------------------------------------------    
     Function CreateWorkspace
     {
         Begin 
         {
             Write-Debug "Starting CreateWorkspace"
             
             Function DeleteWorkspace
             {
                 Write-Verbose "Deleting workspace (if exists): $workspaceName" 
                 $log +=  "Deleting workspace (if exists): $workspaceName" 
                 
                 &$TF workspace /delete  $workspaceName  /noprompt | out-null
 
                 Write-Verbose "Done deleting workspace."
                 $log += "Done deleting workspace."
         }
         Process 
         {
             DeleteWorkspace
         
             if (! (Test-Path $CSRDIR)) 
             {
                 Write-Verbose "Creating folder: $CSRDIR"
                 $log += "Creating folder: $CSRDIR"
                 new-item -itemtype directory -path $CSRDIR -force | out-null
                 Write-Verbose "Completed Creating folder: $CSRDIR" 
                 $log += "Completed Creating folder: $CSRDIR" 
             }
             cd $CSRDIR
 
             Write-Verbose "Creating workspace: $workspaceName"
             $log +=  "Creating workspace: $workspaceName"
 
             &$TF workspace /new /computer:$computerName /server:$TFsServer /noprompt $workspaceName
 
             Write-Verbose "Done Creating workspace: $workspaceName" 
             $log += "Done Creating workspace: $workspaceName" 
 
             Write-Verbose "Getting the latest code from: $TFsServer. This could take awhile..."
             $log += "Getting the latest code from: $TFsServer. This could take awhile..."
             &$TF get $/CSR/CSR /recursive
 
             Write-Verbose "Done getting latest."
             $log += "Done getting latest."
 
             Write-Verbose "Tree initialization is complete."
             $log +=  "Tree initialization is complete."
         }
         End 
         {
             Write-Debug "CreateWorkspace Complete"
         } 
    
    
     #-------------------------------------------------------------------------
     #
     #-------------------------------------------------------------------------
     Function GetNextLabel()
     {
         Begin 
         {
             Write-Debug "Starting GetNextLabel"
         }
         Process
         {
             $major = 1
             $minor = 1
            
             $list = (&$TF labels /format:brief |? { $_ -notmatch "-.+" -and $_ -notmatch "Label.+Owner.+Date"})
             
             if ($list -ne $null) {
                
                 $major,[int]$minor= (($list | Select-Object -last 1).split()).split(".")
             
                 $minor++
                 
                 [int]$major = $major.substring(2)
                 
                 if ($minor -gt 999) {
                     $major++
                     $minor = 1
                 }
             }
             
             $label = "BL{0}.{1:000}" -f $major, $minor
             
             write-output $label
         }
         End
         {
             Write-Debug "GetNextLabel Completed"
         }
     }
     
     #-------------------------------------------------------------------------
     Function MSBuild($outputdir, $webproject, $project, $ref)
     {
         Begin 
         {
             Write-Debug "Starting MSBuild"
             Write-Debug "outputdir: $outputdir webproject: $webproject project: $project ref: $ref"
         }
         Process
         {
             $failed = $false
             &$MSBUILD /p:Configuration=Release  /p:OutDir=$Outputdir /p:WebProjectOutputDir=$webproject $project |% {
                     if ($_ -match "Build FAILED") { $failed = $true } ; $_ 
                 }
             if ($failed) { throw (new-object NullReferenceException) }    
             
             $failed = $false
             &$MSBUILD /p:Configuration=Release /t:ResolveReferences /p:OutDir=$Outputdir\bin\ /p:ReferencePath=$ref $project  |% {
                     if ($_ -match "Build FAILED") { $failed = $true } ; $_ 
                 }
             if ($failed) { throw (new-object NullReferenceException) }    
         }
         End
         {
             Write-Debug "MSBuild Completed"
         }
     }
 
     #-------------------------------------------------------------------------
     #-------------------------------------------------------------------------
     Function ApplyLabel()
     {
         Write-verbose "Create the Label" 
         $log +=  "Create the Label" 
 
         $label = GetNextLabel
             
         &$TF label  $label  $/CSR/CSR /recursive
 
         &$TF get /version:L$($label)
 
         Write-verbose "Applied label $label" 
         $log += "Applied label $label" 
 
         return $Label    
 Process 
 {
     trap [Exception] 
     {
         write-verbose "Build Failed" 
         $log += "Build Failed" 
         exit 1;
     }
     . SetTFS
     . CreateWorkspace
     . ApplyLabel
     
     IF (!$CSRWS -AND !$CSRWEB -AND !$CSRServices)
     {
         Write-Debug "No Options Found Setting ALL"
         $CSRWS = $TRUE
         $CSRWEB = $TRUE
         $CSRServices = $TRUE
     }
     
     Switch ("") 
     {
         {$CSRWEB}
             {
                 Write-Verbose "Building CSRWEB" 
                 $log += "Building CSRWEB" 
                 
                 . MsBuild "$Outdir\CSRWeb\" $WEBPROJECTOUTPUTDIR "$CSRDIR\CSR\CSR\CSRWeb\CSRWeb\CSRWeb.csproj" $REFPATH
                 
 
                 rm  $Outdir\CSRWeb\*.config -recurse
                 rm  $Outdir\CSRWeb\*.pdb -recurse
                 rm  $Outdir\CSRWeb\*.dll -recurse
                 rm  $Outdir\CSRWeb\*.xml -recurse
                 rm  $Outdir\CSRWeb\bin\*.pdb -recurse
                 rm  $Outdir\CSRWeb\bin\*.config -recurse
                 rm  $Outdir\CSRWeb\bin\*.xml -recurse
                 
                 Write-verbose "Build CSRWEB Successful"  
                 $log += "Build CSRWEB Successful"  
             }
         {$CSRWS}
             {
                 Write-verbose "Building CSRWS" 
                 $log +="Building CSRWS" 
         
                 . MsBuild "$Outdir\CSRWS\" $WEBPROJECTOUTPUTDIR1 "$CSRDIR\CSR\CSR\CSRWS\CSRWS\CSRWS.csproj" $REFPATH
                 
 
                 rm  $Outdir\CSRWS\*.config -recurse
                 rm  $Outdir\CSRWS\*.pdb -recurse
                 rm  $Outdir\CSRWS\*.dll -recurse
                 rm  $Outdir\CSRWS\*.xml -recurse
                 rm  $Outdir\CSRWS\bin\*.pdb -recurse
                 rm  $Outdir\CSRWS\bin\*.config -recurse
                 rm  $Outdir\CSRWS\bin\*.xml -recurse
                
                 Write-verbose "Build CSRWS Successful" 
                 $log += "Build CSRWS Successful" 
             }
         {$CSRServices}
             {
                 Write-verbose "Building CSR Services"
                 $Log += "Building CSR Services"
             
                 . MsBuild "$Outdir\CSRSERVICES\" $WEBPROJECTOUTPUTDIR2 "$CSRDIR\CSR\CSR\CSRSERVICES\CSRSERVICES\CSRSERVICES.csproj" $REFPATH
                 
 
                 rm  $Outdir\CSRSERVICES\*.config -recurse
                 rm  $Outdir\CSRSERVICES\*.pdb -recurse
                 rm  $Outdir\CSRSERVICES\*.dll -recurse
                 rm  $Outdir\CSRSERVICES\*.xml -recurse
                 rm  $Outdir\CSRSERVICES\bin\*.pdb -recurse
                 rm  $Outdir\CSRSERVICES\bin\*.config -recurse
                 rm  $Outdir\CSRSERVICES\bin\*.xml -recurse
                 
                 Write-verbose "Build CSR Services Successful"
                 $Log += "Build CSR Services Successful"
             }
 End 
 {
     IF ($LogPath) 
     {
         $log | Out-file -FilePath $LogPath -Encoding ascii -Append
     }
`

