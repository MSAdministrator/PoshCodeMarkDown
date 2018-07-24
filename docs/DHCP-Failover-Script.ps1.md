---
Author: vocatus
Publisher: 
Copyright: 
Email: 
Version: 1.1
Encoding: ascii
License: cc0
PoshCode ID: 3562
Published Date: 2012-08-09t15
Archived Date: 2012-08-18t03
---

# dhcp failover script - 

## Description

dhcp failover/watchdog script.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 :: Purpose:         DHCP server Watchdog & Failover script. Read notes below
 :: Requirements:    1. Domain administrator credentials & "Logon as a batch job" rights
 ::                  2. Proper firewall configuration to allow connection
 ::                  3. Proper permissions on the DHCP backup directory
 :: Author:          vocatus
 :: Version:         1.1b - Changed DATE to CUR_DATE format to be consistent with all other scripts
 ::                  1.1 - Comments work
 ::                      - Tuned some parameters (ping count on checking)
 ::                      - Some logging tweaks
 ::                      - Renamed FAILOVER_DELAY to FAILOVER_RECHECK_DELAY for clarity
 ::                  1.0d Some logging tweaks
 ::                  1.0c Some logging tweaks
 ::                  1.0 Initial write
 :: Notes:           I wrote this script after failing to find a satisfactory method of performing
 ::                  watchdog/failover between two Windows Server 2008 R2 DHCP servers.
 ::                 
 :: Use:             This script has two modes: "Watchdog" and "Failover." 
 ::                  - Watchdog checks the status of the remote DHCP service, logs it, and then grabs the remote DHCP db backup file and imports it.
 ::                  - Failover mode is activated when the script cannot determine the status of the remote DHCP server. The script then activates 
 ::                    the local DHCP server with the latest backup copy it successfully retrieved from the primary server.
 ::                  
 :: Instructions: 
 ::                  1. Tune the variables in this script to your desired backup location and frequency
 ::                  2. On the primary server: set the DHCP backup interval to your desired backup frequency. The value is in minutes; I recommend 5 minutes.
 ::                     You do this by modifying this registry key: HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\DHCPServer\Parameters\BackupInterval
 ::                  3. On the backup server:  set this script to run as a scheduled task. I recommend every 10 minutes. 
 :: Notice: 
 ::                 !! Make sure to set it only to run if it isn't already running! If there is a failover you could have 
 ::                    Task Scheduler spawn a new instance of the script every n minutes and end up with hundreds of copies
 ::                    of this script running.
 
 
 :: Prep
 SETLOCAL
 @echo off
 cls
 set VERSION=1.1b
 title [DHCP Server Watchdog v%VERSION%]
 
 
 :::::::::::::::
 :: Variables :: - Set these. Do not use trailing slashes (\) in directory names (this is important!).
 :::::::::::::::
 
 :: Remote server is the PRIMARY DHCP server we're watching. Use a hostname or IP address.
 set REMOTE_SERVER=mikado
 
 :: Location of the DHCP backup on the primary server
 :: Best practice is to leave this alone, unless you have a custom backup location.
 :: The script builds the backup line like this: \\%REMOTE_SERVER%\c$\%REMOTE_BACKUP_PATH%
 set REMOTE_BACKUP_PATH=Windows\system32\dhcp\backup
 
 :: Location of your backup/standby file. I normally copy directly to my backup server's DHCP directory. 
 :: The script builds the local backup line like this: c:\windows\system32\dhcp\[backup folders]
 set LOCAL_BACKUP_PATH=%SystemRoot%\system32\dhcp
 
 :: When a failover is triggered, how many seconds should we wait in between each attempt to contact the primary server again?
 set FAILOVER_RECHECK_DELAY=15
 
 :: Log options. Don't put an extension on the log file name. (Important!) The script sets this later on.
 set LOGPATH=%SystemDrive%\Logs
 set LOGFILE=%COMPUTERNAME%_DHCP_watchdog
 
 :: Max log file size allowed (in bytes) before rotation and archive. I recommend setting this to 2 MB (2097152).
 :: Example: 524288 is half a megabyte (~500KB)
 set LOG_MAX_SIZE=2097152
 
 :: \/ Don't touch anything below this line. If you do, you will break something.
 set CUR_DATE=%DATE:~-4%-%DATE:~4,2%-%DATE:~7,2%
 
 
 :::::::::::::::::::::::
 :: LOG FILE HANDLING :: - This section handles the log file
 :::::::::::::::::::::::
 
 :: Make the logfile if it doesn't exist
 if not exist %LOGPATH% mkdir %LOGPATH%
 if not exist %LOGPATH%\%LOGFILE%.log goto new_log
 
 :: Check log size. If it hasn't exceeded our size limit, jump straight to Watchdog mode
 for %%R in (%LOGPATH%\%LOGFILE%.log) do if %%~zR LSS %LOG_MAX_SIZE% goto newrun
 
 :: However, if the log was too big, go ahead and rotate it.
 pushd %LOGPATH%
 del %LOGFILE%.ancient 2>NUL
 rename %LOGFILE%.oldest %LOGFILE%.ancient 2>NUL
 rename %LOGFILE%.older %LOGFILE%.oldest 2>NUL
 rename %LOGFILE%.old %LOGFILE%.older 2>NUL
 rename %LOGFILE%.log %LOGFILE%.old 2>NUL
 popd
 
 :: And then create the header for the new log file
 :new_log
 echo ------------------------------------------------------------------------------------->> %LOGPATH%\%LOGFILE%.log 2>&1
 echo  Initializing new DHCP Server Watchdog log on %CUR_DATE% at %TIME%, max log size %LOG_MAX_SIZE% bytes>> %LOGPATH%\%LOGFILE%.log 2>&1
 echo ------------------------------------------------------------------------------------->> %LOGPATH%\%LOGFILE%.log 2>&1
 echo.>> %LOGPATH%\%LOGFILE%.log 2>&1
 
 :: New run section - if we just launched the script, write a header for this run
 :newrun
 echo ------------------------------------------------------------------------------------->> %LOGPATH%\%LOGFILE%.log 2>&1
 echo  DHCP Server Watchdog v%VERSION%, %CUR_DATE%>> %LOGPATH%\%LOGFILE%.log 2>&1
 echo   Running as %USERDOMAIN%\%USERNAME% on %COMPUTERNAME%>> %LOGPATH%\%LOGFILE%.log 2>&1
 echo.>> %LOGPATH%\%LOGFILE%.log 2>&1
 echo  Job Options>> %LOGPATH%\%LOGFILE%.log 2>&1
 echo   Log location:            %LOGPATH%\%LOGFILE%.log>> %LOGPATH%\%LOGFILE%.log 2>&1
 echo   Log max size:            %LOG_MAX_SIZE% bytes>> %LOGPATH%\%LOGFILE%.log 2>&1
 echo   Watching primary server: %REMOTE_SERVER%>> %LOGPATH%\%LOGFILE%.log 2>&1
 echo   Mirroring this DHCP db:  %REMOTE_BACKUP_PATH%>> %LOGPATH%\%LOGFILE%.log 2>&1
 echo   Local backup location:   %LOCAL_BACKUP_PATH%>> %LOGPATH%\%LOGFILE%.log 2>&1
 echo ------------------------------------------------------------------------------------->> %LOGPATH%\%LOGFILE%.log 2>&1
 echo %TIME%         Starting Watchdog mode.>> %LOGPATH%\%LOGFILE%.log 2>&1
 echo.
 echo  DHCP Server Watchdog v%VERSION%
 echo   Running as: %USERDOMAIN%\%USERNAME% on %COMPUTERNAME%
 echo   Log:        %LOGPATH%\%LOGFILE%.log
 
 
 :::::::::::::::::::
 :: WATCHDOG MODE ::
 :::::::::::::::::::
 
 :watchdog
 
 :: Ping the server to see if it's up
 echo.
 echo   Verifying proper operation of DHCP server on %REMOTE_SERVER%, please wait...
 echo.
 echo %TIME%         Pinging %REMOTE_SERVER%...>> %LOGPATH%\%LOGFILE%.log 2>&1
 echo %TIME%         Pinging %REMOTE_SERVER%...
 ping %REMOTE_SERVER% -n 2 >NUL 2>&1
 if %ERRORLEVEL%==1 echo %TIME% WARNING %REMOTE_SERVER% failed to respond to ping. && echo %TIME% WARNING %REMOTE_SERVER% failed to respond to ping.>> %LOGPATH%\%LOGFILE%.log 2>&1
 if not %ERRORLEVEL%==1 echo %TIME% SUCCESS %REMOTE_SERVER% responded to ping. && echo %TIME% SUCCESS %REMOTE_SERVER% responded to ping.>> %LOGPATH%\%LOGFILE%.log 2>&1
 
 :: Check & Log
 echo %TIME%         Checking DHCP server status on %REMOTE_SERVER%...>> %LOGPATH%\%LOGFILE%.log 2>&1
 echo %TIME%         Checking DHCP server status on %REMOTE_SERVER%...
 
 :: Reset ERRORLEVEL back to 0
 ver > NUL
 
 :: Use "SC" to check the status of "Dhcpserver" service, find the "RUNNING" state, and act accordingly based on the return code
 sc \\%REMOTE_SERVER% query Dhcpserver | find "RUNNING" >NUL 2>&1
 if %ERRORLEVEL%==0 echo %TIME% SUCCESS The DHCP service is running on %REMOTE_SERVER%.>> %LOGPATH%\%LOGFILE%.log 2>&1
 if %ERRORLEVEL%==0 echo %TIME% SUCCESS The DHCP service is running on %REMOTE_SERVER%.
 
 :: This section only executes if the test failed.
 if not %ERRORLEVEL%==0 ( 
 	echo %TIME% FAILURE The DHCP service is not running on %REMOTE_SERVER%.>> %LOGPATH%\%LOGFILE%.log 2>&1
 	echo %TIME%         Activating failover procedure. Local DHCP server will be initialized using most recent successful backup.>> %LOGPATH%\%LOGFILE%.log 2>&1
 	echo %TIME% FAILURE The DHCP service is not running on %REMOTE_SERVER%.
 	echo %TIME%         Activating failover procedure. Local DHCP server will be initialized using most recent successful backup.
 	goto failover
 	)
 
 :: Reset ERRORLEVEL back to 0
 ver > NUL
 
 :: Fetch
 echo %TIME%         Fetching DHCP database backup from %REMOTE_SERVER%...>> %LOGPATH%\%LOGFILE%.log 2>&1
 echo %TIME%         Fetching DHCP database backup from %REMOTE_SERVER%...
 xcopy \\%REMOTE_SERVER%\c$\%REMOTE_BACKUP_PATH%\* %LOCAL_BACKUP_PATH%\backup_new_pending\ /E /Y /Q >NUL
 
 :: If the copy SUCCEEDED, this executes
 if %ERRORLEVEL%==0 ( 
 	echo %TIME% SUCCESS Backup fetched from %REMOTE_SERVER%.>> %LOGPATH%\%LOGFILE%.log 2>&1
 	echo %TIME% SUCCESS Backup fetched from %REMOTE_SERVER%.
 	echo %TIME%         Rotating database backups...>> %LOGPATH%\%LOGFILE%.log 2>&1
 	echo %TIME%         Rotating database backups...
 	:: Rotate backups and use newest copy
 	rmdir /S /Q %LOCAL_BACKUP_PATH%\backup5
 	if exist %LOCAL_BACKUP_PATH%\backup4 move /Y %LOCAL_BACKUP_PATH%\backup4 %LOCAL_BACKUP_PATH%\backup5 2>&1
 	if exist %LOCAL_BACKUP_PATH%\backup3 move /Y %LOCAL_BACKUP_PATH%\backup3 %LOCAL_BACKUP_PATH%\backup4 2>&1
 	if exist %LOCAL_BACKUP_PATH%\backup2 move /Y %LOCAL_BACKUP_PATH%\backup2 %LOCAL_BACKUP_PATH%\backup3 2>&1
 	if exist %LOCAL_BACKUP_PATH%\backup move /Y %LOCAL_BACKUP_PATH%\backup %LOCAL_BACKUP_PATH%\backup2 2>&1
 	move /Y %LOCAL_BACKUP_PATH%\backup_new_pending %LOCAL_BACKUP_PATH%\backup >NUL
 	echo %TIME%         Database backups rotated.>> %LOGPATH%\%LOGFILE%.log 2>&1
 	echo %TIME%         Database backups rotated.
 	)
 
 :: If the copy FAILED, this executes:
 if not %ERRORLEVEL%==0 ( 
 	echo %TIME% WARNING There was an error copying the backup from %REMOTE_SERVER%.>> %LOGPATH%\%LOGFILE%.log 2>&1
 	echo %TIME%         You may want to look into this since we were able to check the DHCPserver service status but the file copy failed.>> %LOGPATH%\%LOGFILE%.log 2>&1
 	echo %TIME%         Skipping new database import due to copy failure.>> %LOGPATH%\%LOGFILE%.log 2>&1
 	echo %TIME%         Job complete with errors.>> %LOGPATH%\%LOGFILE%.log 2>&1
 	echo %TIME% WARNING There was an error copying the backup from %REMOTE_SERVER%.
 	echo %TIME%         You may want to look into this since we were able to check the DHCPserver service status but the file copy failed.
 	echo %TIME%         Skipping new database import due to copy failure.
 	echo %TIME%         Job complete with errors.
 	)
 	
 :: Import database
 ver > NUL
 echo %TIME%         Starting local DHCP server to import new database...>> %LOGPATH%\%LOGFILE%.log 2>&1
 echo %TIME%         Starting local DHCP server to import new database...
 	net start Dhcpserver 2>&1
 echo %TIME%         Local DHCP server running. Performing import...>> %LOGPATH%\%LOGFILE%.log 2>&1
 echo %TIME%         Local DHCP server running. Performing import...
 	netsh dhcp server restore %LOCAL_BACKUP_PATH%\backup
 echo %TIME%         Import complete.>> %LOGPATH%\%LOGFILE%.log 2>&1
 echo %TIME%         Import complete.
 echo %TIME%         Stopping local DHCP server...>> %LOGPATH%\%LOGFILE%.log 2>&1
 echo %TIME%         Stopping local DHCP server...
 	net stop Dhcpserver 2>&1
 echo %TIME%         Local DHCP server stopped.>> %LOGPATH%\%LOGFILE%.log 2>&1
 echo %TIME%         Local DHCP server stopped.
 echo %TIME% SUCCESS Job complete, DHCP database backed up and ready for use. Exiting.>> %LOGPATH%\%LOGFILE%.log 2>&1
 echo %TIME% SUCCESS Job complete, DHCP database backed up and ready for use. Exiting.
 goto EOF
 
 
 :::::::::::::::::::
 :: FAILOVER MODE ::
 :::::::::::::::::::
 
 :failover
 :: Log this AND display to console
 echo %TIME% WARNING Failover activated.>> %LOGPATH%\%LOGFILE%.log 2>&1
 echo %TIME%         Starting local DHCP server using most recent successful backup...>> %LOGPATH%\%LOGFILE%.log 2>&1
 echo.
 echo %TIME% WARNING Could not contact primary DHCP server "%REMOTE_SERVER%." Failover activated.
 echo %TIME%         Starting local DHCP server using most recent successful backup...
 echo.
 net start Dhcpserver
 echo %TIME%         Local DHCP server started.>> %LOGPATH%\%LOGFILE%.log 2>&1
 echo %TIME%         Entering monitoring loop. Checking if %REMOTE_SERVER% is back up every %FAILOVER_RECHECK_DELAY% seconds...>> %LOGPATH%\%LOGFILE%.log 2>&1
 echo %TIME%         Local DHCP server started.
 echo %TIME%         Entering monitoring loop. Checking if %REMOTE_SERVER% is back up every %FAILOVER_RECHECK_DELAY% seconds...
 
 
 :failover_loop
 :: First we ping the server
 ver >NUL
 ping %REMOTE_SERVER% -n 3 >NUL 2>&1
 :: If no ping response, this section executes
 if %ERRORLEVEL%==1 (
 	echo %TIME% FAILURE No ping response from %REMOTE_SERVER%. Waiting %FAILOVER_RECHECK_DELAY% seconds to check again.>> %LOGPATH%\%LOGFILE%.log 2>&1
 	echo %TIME% FAILURE No ping response from %REMOTE_SERVER%. Waiting %FAILOVER_RECHECK_DELAY% seconds to check again.
 	ping localhost -n %FAILOVER_RECHECK_DELAY% >NUL 2>&1
 	goto failover_loop
 	)
 
 :: If yes ping response, this section executes
 :: This declaration is required to get the nested IF ERRORLEVEL test to function correctly
 SETLOCAL ENABLEDELAYEDEXPANSION
 if not %ERRORLEVEL%==1 (
 	echo %TIME% NOTICE %REMOTE_SERVER% is responding to pings.>> %LOGPATH%\%LOGFILE%.log 2>&1
 	echo %TIME% NOTICE %REMOTE_SERVER% is responding to pings.
 	echo %TIME%        Checking DHCP server status on %REMOTE_SERVER%...>> %LOGPATH%\%LOGFILE%.log 2>&1
 	echo %TIME%        Checking DHCP server status on %REMOTE_SERVER%...
 	
 	:: This section checks to see if the Dhcpserver service is back up and acts accordingly
 	sc \\%REMOTE_SERVER% query Dhcpserver | find "RUNNING" >NUL 2>&1
 		:: The exclamation points around ERRORLEVEL here prevent it from incorrectly being expanded using the external ERRORLEVEL results from the first IF statement
 		if !ERRORLEVEL!==0 (
 				echo %TIME% SUCCESS The DHCP service is running on %REMOTE_SERVER%.>> %LOGPATH%\%LOGFILE%.log 2>&1
 				echo %TIME% SUCCESS The DHCP service is running on %REMOTE_SERVER%.
 				echo %TIME%         The primary DHCP server %REMOTE_SERVER% is back up. Stopping local DHCP service...>> %LOGPATH%\%LOGFILE%.log 2>&1
 				echo %TIME%         The primary DHCP server %REMOTE_SERVER% is back up. Stopping local DHCP service...
 				net stop Dhcpserver
 				echo %TIME%         Recovery complete. Exiting.>> %LOGPATH%\%LOGFILE%.log 2>&1
 				echo %TIME%         Recovery complete. Exiting.
 				goto EOF
 				)
 	)
 ENDLOCAL
 
 :: If the host responds to pings but the DHCP service isn't running, this executes
 echo %TIME% FAILURE %REMOTE_SERVER% is responding to pings, but DHCP isn't responding (yet?). Will try again in %FAILOVER_RECHECK_DELAY% seconds.>> %LOGPATH%\%LOGFILE%.log 2>&1
 echo %TIME% FAILURE %REMOTE_SERVER% is responding to pings, but DHCP isn't responding (yet?). Will try again in %FAILOVER_RECHECK_DELAY% seconds.
 ver >NUL
 goto failover_loop
 
 ENDLOCAL
 echo.>> %LOGPATH%\%LOGFILE%.log 2>&1
 :EOF
`

