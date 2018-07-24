---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5713
Published Date: 2016-01-25t19
Archived Date: 2016-09-09t23
---

# get-logonsessions - 

## Description

depends on “steroid” library (see https

## Comments



## Usage



## TODO



## function

`get-logonsessions`

## Code

`#
 #
 function Get-LogonSessions {
   <#
     .SYNOPSIS
         Describes the logon session or sessions associated with a user logged (
         instead Win32_LogonSession class).
     .NOTES
         Author: greg zakharov
   #>
   
   begin {
     [accelerators]::Add('marshal', [Runtime.InteropServices.Marshal])
     
     $LUID = struct LUID {
       UInt32 'LowPart';
       UInt32 'HighPart';
     }
     
     struct LSA_UNICODE_STRING {
       UInt16 'Length';
       UInt16 'MaximumLength';
       IntPtr 'Buffer';
     } | Out-Null
     
     struct LSA_LAST_INTER_LOGON_INFO {
       Int64  'LastSuccessfulLogon';
       Int64  'LastFailedLogon';
       UInt32 'FailedAttemptCountSinceLastSuccessfulLogon';
     } | Out-Null
     
     enum SECURITY_LOGON_TYPE UInt32 {
       Interactive             = 2
       Network                 = 3
       Batch                   = 4
       Serivce                 = 5
       Proxy                   = 6
       Unlock                  = 7
       NetworkCleartext        = 8
       NewCredentials          = 9
       RemoteInteractive       = 10
       CachedInteractive       = 11
       CachedRemoteInteractive = 12
       CachedUnlcok            = 13
     } | Out-Null
     
     $slsd = struct SECURITY_LOGON_SESSION_DATA {
       UInt32                    'Size';
       LUID                      'LogonId';
       LSA_UNICODE_STRING        'UserName';
       LSA_UNICODE_STRING        'LogonDomain';
       LSA_UNICODE_STRING        'AuthenticationPackage';
       SECURITY_LOGON_TYPE       'LogonType';
       UInt32                    'Session';
       IntPtr                    'Sid';
       Int64                     'LogonTime';
       LSA_UNICODE_STRING        'LogonServer';
       LSA_UNICODE_STRING        'DnsDomainName';
       LSA_UNICODE_STRING        'Upn';
       LSA_LAST_INTER_LOGON_INFO 'LastLogonInfo';
       LSA_UNICODE_STRING        'LogonScript';
       LSA_UNICODE_STRING        'ProfilePath';
       LSA_UNICODE_STRING        'HomeDirectory';
       LSA_UNICODE_STRING        'HomeDirectoryDrive';
       Int64                     'LogoffTime';
       Int64                     'KickOffTime';
       Int64                     'PassworkLastSet';
       Int64                     'PasswordCanChange';
       Int64                     'PasswordMustChange';
     }
     
     $LsaFreeReturnBuffer = delegate secur32.dll LsaFreeReturnBuffer Int32 @(
       [IntPtr]
     )
     $LsaEnumerateLogonSessions = delegate secur32.dll LsaEnumerateLogonSessions Int32 @(
       [UInt32].MakeByRefType(), [IntPtr].MakeByRefType()
     )
     $LsaGetLogonSessionData = delegate secur32.dll LsaGetLogonSessionData Int32 @(
       [IntPtr], [IntPtr].MakeByRefType()
     )
     
     $STATUS_SUCCESS       = 0x00000000
     $STATUS_ACCESS_DENIED = 0xC0000022
   }
   process {
     try {
       
       if (($ret = $LsaEnumerateLogonSessions.Invoke([ref]$count, [ref]$first)) -ne $STATUS_SUCCESS) {
         throw (New-Object Exception($('Could not enumerate logon sessions. Error 0x{0:X}' -f $ret)))
       }
       
       $str = $slsd
       for ($i = 0; $i -lt $count; $i++) {
         [IntPtr]$out = [IntPtr]::Zero
         if ($LsaGetLogonSessionData.Invoke($ptr, [ref]$out) -eq $STATUS_ACCESS_DENIED) {
           "You have $(if(!(IsAdmin)){'not'}) administrator rights."
           break
         }
         
         $str = $out -as $str
         $SECURITY_LOGON_SESSION_DATA = New-Object PSObject -Property @{
           UserName       = ($str.LogonDomain.Buffer, $str.UserName.Buffer | % {
             [Marshal]::PtrToStringUni($_)
           }) -join '\'
           LogonType      = $str.LogonType
           Session        = $str.Session
           SID            = $(try{New-Object Security.Principal.SecurityIdentifier($str.Sid)}catch{})
           Authentication = [Marshal]::PtrToStringUni($str.AuthenticationPackage.Buffer)
           LogonTime      = [DateTime]::FromFileTime($str.LogonTime)
         }
         $SECURITY_LOGON_SESSION_DATA.PSObject.TypeNames.Insert(0, 'SECURITY_LOGON_SESSION_DATA')
         $SECURITY_LOGON_SESSION_DATA
         
         $ptr = [IntPtr]($ptr.ToInt64() + $LUID::GetSize())
         [void]$LsaFreeReturnBuffer.Invoke($out)
         $str = $slsd
     }
     catch { $_.Exception }
     finally {
       [void]$LsaFreeReturnBuffer.Invoke($first)
     }
   }
   end {
     [void][accelerators]::Remove('marshal')
   }
 }
`

