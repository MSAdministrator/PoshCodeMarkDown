---
Author: ernst schlee
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6654
Published Date: 2017-12-19t12
Archived Date: 2017-01-01t19
---

# add-type extends - 

## Description

arise you ever a wish to call cpython code via add-type cmdlets? now you have this ability! note that you need “morphine” library (https

## Comments



## Usage



## TODO



## function

`get-logonsessions`

## Code

`#
 #
 function Get-LogonSessions {
   Set-Content function:Add-Type (Unlock-Python)
 Add-Type -Language Python -TypeDefinition @'
 from calendar import timegm
 from ctypes   import (
    POINTER, Structure, byref, cast, c_long, c_ulong, c_longlong,
    c_ushort, c_void_p, c_wchar_p, sizeof, windll
 )
 from datetime import datetime, timedelta
 from enum     import IntEnum
 from sys      import exit
 
 SECURITY_LOGON_TYPE = IntEnum('SECURITY_LOGON_TYPE', ' \
    Unknown Interactive Network Batch Service Proxy Unlock \
    NetworkCleartext NewCredentials RemoteInteractive \
    CachedInteractive CachedRemoteInteractive CachedUnlock \
 ')
 
 class LUID(Structure):
    _fields_ = [
       ('LowPart',  c_ulong),
       ('HighPart', c_long),
    ]
 
 class LSA_UNICODE_STRING(Structure):
    _fields_ = [
       ('Length',        c_ushort),
       ('MaximumLength', c_ushort),
       ('Buffer',        c_wchar_p),
    ]
 
 class LSA_LAST_INTER_LOGON_INFO(Structure):
    _fields_ = [
       ('LastSuccessfulLogon',                        c_longlong),
       ('LastFailedLogon',                            c_longlong),
       ('FailedAttemptCountSinceLastSuccessfulLogon', c_ulong),
    ]
 
 class SECURITY_LOGON_SESSION_DATA(Structure):
    _fields_ = [
       ('Size',                  c_ulong),
       ('LogonId',               LUID),
       ('UserName',              LSA_UNICODE_STRING),
       ('LogonDomain',           LSA_UNICODE_STRING),
       ('AuthenticationPackage', LSA_UNICODE_STRING),
       ('_LogonType',            c_ulong),
       ('Session',               c_ulong),
       ('_Sid',                  c_void_p),
       ('_LogonTime',            c_longlong),
       ('LogonServer',           LSA_UNICODE_STRING),
       ('DnsDomainName',         LSA_UNICODE_STRING),
       ('Upn',                   LSA_UNICODE_STRING),
       ('UserFlags',             c_ulong),
       ('LastLogonInfo',         LSA_LAST_INTER_LOGON_INFO),
       ('LogonScript',           LSA_UNICODE_STRING),
       ('ProfilePath',           LSA_UNICODE_STRING),
       ('HomeDirectory',         LSA_UNICODE_STRING),
       ('HomeDirectoryDrive',    LSA_UNICODE_STRING),
       ('LogoffTime',            c_longlong),
       ('KickOffTime',           c_longlong),
       ('PasswordLastSet',       c_longlong),
       ('PasswordCanChange',     c_longlong),
       ('PasswordMustChange',    c_longlong),
    ]
    @property
    def LogonType(self):
       return SECURITY_LOGON_TYPE(self._LogonType).name if self._LogonType else None
    @property
    def Sid(self):
       ptr = c_wchar_p()
       if windll.advapi32.ConvertSidToStringSidW(self._Sid, byref(ptr)):
          sid = ptr.value
          windll.kernel32.LocalFree(ptr)
          return sid
    @property
    def LogonTime(self):
       lt = datetime(1601, 1, 1) + timedelta(microseconds=self._LogonTime / 10)
       return datetime.fromtimestamp(timegm(lt.timetuple()))
 
 def printlsaerror(nts):
    msg = c_void_p()
    windll.kernel32.FormatMessageW(
       0x00001100, None, windll.advapi32.LsaNtStatusToWinError(nts),
       1024, byref(msg), 0, None
    )
    err = c_wchar_p(msg.value).value
    windll.kernel32.LocalFree(msg)
    print(err.strip() if err else 'Unknown error has been occured.')
 
 if __name__ == '__main__':
    cnt, lsl = c_ulong(), c_void_p()
    nts = windll.secur32.LsaEnumerateLogonSessions(byref(cnt), byref(lsl))
    if nts:
       printlsaerror(nts)
       exit(1)
    itr = c_void_p(lsl.value)
    for i in range(0, cnt.value):
       lsd = c_void_p()
       nts = windll.secur32.LsaGetLogonSessionData(itr, byref(lsd))
       if nts:
          printlsaerror(nts)
          break
       slsd = cast(lsd, POINTER(SECURITY_LOGON_SESSION_DATA)).contents
       print('Logon type   : %s' % slsd.LogonType)
       print('User name    : %s\\%s' % (slsd.LogonServer.Buffer, slsd.UserName.Buffer))
       print('Auth package : %s' % slsd.AuthenticationPackage.Buffer)
       print('Session      : %u' % slsd.Session)
       print('Sid          : %s' % slsd.Sid)
       print('Logon time   : %s\n' % slsd.LogonTime)
       itr = c_void_p(itr.value + sizeof(LUID))
       windll.secur32.LsaFreeReturnBuffer(lsd)
    windll.secur32.LsaFreeReturnBuffer(lsl)
 '@
 }
`

