---
Author: scripty harry
Publisher: 
Copyright: 
Email: 
Version: 6.5.5
Encoding: ascii
License: cc0
PoshCode ID: 6583
Published Date: 2016-10-18t11
Archived Date: 2016-12-22t04
---

# netbackup backup report - nbu-backupreport.ps1

## Description

script to create a backup report from netbackup jobs

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ###################################################################################################
 #
 #
 #
 #
 #
 #
 #
 ###################################################################################################
 #
 #
 {
  {
  }
 }
 Clear-Host
 [console]::ForegroundColor = "White"
 Write-Host ""
 Write-Host "============================================================" -ForegroundColor "Yellow"
 Write-Host "                  NetBackup - Daily Report"
 Write-Host "============================================================" -ForegroundColor "Yellow"
 Write-Host ""
 Write-Host ""
 {
 }
 {
 }
 {
  {
  }
  Else
  {
   $PolicyFilter = $Policy
  }
  {
   ForEach ($ClientStr in $PolicyClientList)
   {
    {
    }
    #-- Start Statistics gathering for the currently processing policy and the currently processing client
    {
     {
 	}
 	{
 	}
     If (($TotalKB -ge 1048576) -AND ($TotalKB -lt 1073741824))
 	{
 	}
 	If (($TotalKB -ge 1073741824) -AND ($TotalKB -lt 1099511627776))
  	{
 	}
 	If ($TotalKB -ge 1099511627776)
     {
     }
    }
    {
     {
      {
      }
      Else
      {
      }
 	 {
 	 }
 	 {
 	 }
 	 Switch ($StatusCode)
 	 {
 	  0
 	  {
 	  }
 	  1
 	  {
 	  }
 	  Default
 	  {
 	  }
 	 }
 	 {
 	  {
 	   {
 	   }
 	   {
 	   }
 	   Else
 	   {
 	   }
 	   {
 	   }
 	   {
 	   }
 	  }
 	  {
 	   {
 	   }
 	   Else
 	   {
 	    {
 	    }
 	    Else
 	    {
 	    }
 	   }
 	   {
 	   }
 	   {
 	   }
 	  }
 	  {
 	   If ($JobFailCount -eq 1)
 	   {
 	   }
 	   Else
 	   {
 	    {
 	    }
 	    Else
 	    {
 	    }
 	   }
 	   {
 	   }
 	   {
 	   }
 	  }
 	 }
 	}
    }
    {
 	{
 	}
 	{
      Continue
 	}
    }
   }
   {
   }
   {
   }
   {
   }
  }
 }
 {
 }
 If (($GrandTotalKB -gt 1048576) -AND ($GrandTotalKB -lt 1073741824))
 {
 }
 If ($GrandTotalKB -gt 1073741824 -AND ($GrandTotalKB -lt 1099511627776))
 {
 }
 If ($GrandTotalKB -gt 1099511627776)
 {
 }
 {
  {
  }
 }
 {
 }
 {
  Try
  {
  }
  Catch
  {
  }
  {
   {
   }
  }
 }
 {
 }
 {
 }
 {
  {
  }
 }
 {
 }
 {
 }
 {
 }
 {
 }
 {
 }
 [console]::ResetColor()
 
  End of Script
  Below is an example of the actual output
 
 Report Generated on NBUSRV, monday 06-12-2010 8:30:07
 
 Backup report for friday 03-12-2010
 
 Failed jobs during the FileBackup backup
 ----------------------------------------
 STATUS CLIENT        POLICY           SCHED      SERVER      TIME COMPLETED
 156    SRV001        FileBackup       Week       NBUSRV      12/04/2010 15:17:48 
 
 Failed jobs during the Linux_filebackup2 backup
 -----------------------------------------------
 STATUS CLIENT        POLICY           SCHED      SERVER      TIME COMPLETED
  71    SRV002        Linux_filebackup Week       NBUSRV      12/03/2010 18:17:54 
 
 Partially Successful jobs during the FileBackup backup
 ------------------------------------------------------
 STATUS CLIENT        POLICY           SCHED      SERVER      TIME COMPLETED
   1    NBUSRV        FileBackup       Week       NBUSRV      12/03/2010 23:56:46 
 
 Partially Successful jobs during the FileBackup2 backup
 -------------------------------------------------------
 STATUS CLIENT        POLICY           SCHED      SERVER      TIME COMPLETED
   1    SRV003        FileBackup2      Week       NBUSRV      12/03/2010 20:44:03 
 
 Partially Successful jobs during the NAS_FileBackup_SQL_WWW backup
 ------------------------------------------------------------------
 STATUS CLIENT        POLICY           SCHED      SERVER      TIME COMPLETED
   1    SRV004        NAS_FileBackup_S Full       NBUSRV      12/04/2010 12:26:59 
   1    SRV005        NAS_FileBackup_S Full       NBUSRV      12/04/2010 00:02:49 
 
 Partially Successful jobs during the PRTG backup
 ------------------------------------------------
 STATUS CLIENT        POLICY           SCHED      SERVER      TIME COMPLETED
   1    SRV006        PRTG             Full       NBUSRV      12/04/2010 13:28:54 
 
 Successful jobs during the CatalogBackup backup
 -----------------------------------------------
 STATUS CLIENT        POLICY           SCHED      SERVER      TIME COMPLETED
   0    NBUSRV        CatalogBackup    Daily      NBUSRV      12/06/2010 08:11:56 
 
 Successful jobs during the ExchangeBackup backup
 ------------------------------------------------
 STATUS CLIENT        POLICY           SCHED      SERVER      TIME COMPLETED
   0    SRV003        ExchangeBackup   Week       NBUSRV      12/04/2010 19:51:29 
 
 Successful jobs during the FileBackup backup
 --------------------------------------------
 STATUS CLIENT        POLICY           SCHED      SERVER      TIME COMPLETED
   0    SRV007        FileBackup       Week       NBUSRV      12/04/2010 14:21:31 
   0    SRV008        FileBackup       Week       NBUSRV      12/03/2010 20:54:54 
   0    SRV009        FileBackup       Week       NBUSRV      12/04/2010 01:40:42 
   0    SRV010        FileBackup       Week       NBUSRV      12/04/2010 01:33:09 
   0    SRV004        FileBackup       Week       NBUSRV      12/03/2010 23:57:03 
   0    SRV011        FileBackup       Week       NBUSRV      12/04/2010 01:16:40 
   0    SRV012        FileBackup       Week       NBUSRV      12/04/2010 01:52:23 
   0    SRV013        FileBackup       Week       NBUSRV      12/04/2010 18:11:10 
   0    SRV014        FileBackup       Week       NBUSRV      12/04/2010 01:17:05 
   0    SRV015        FileBackup       Week       NBUSRV      12/04/2010 01:14:31 
   0    SRV016        FileBackup       Week       NBUSRV      12/04/2010 13:20:02 
   0    SRV017        FileBackup       Week       NBUSRV      12/04/2010 01:36:57 
   0    SRV005        FileBackup       Week       NBUSRV      12/04/2010 11:25:06 
 
 Successful jobs during the FileBackup2 backup
 ---------------------------------------------
 STATUS CLIENT        POLICY           SCHED      SERVER      TIME COMPLETED
   0    SRV018        FileBackup2      Week       NBUSRV      12/05/2010 14:49:59 
 
 Successful jobs during the Linux_filebackup backup
 --------------------------------------------------
 STATUS CLIENT        POLICY           SCHED      SERVER      TIME COMPLETED
   0    SRV019        Linux_filebackup Week       NBUSRV      12/03/2010 18:09:46 
   0    SRV020        Linux_filebackup Week       NBUSRV      12/03/2010 20:55:21 
   0    SRV021        Linux_filebackup Week       NBUSRV      12/03/2010 18:16:15 
 
 Successful jobs during the MailboxBackup backup
 -----------------------------------------------
 STATUS CLIENT        POLICY           SCHED      SERVER      TIME COMPLETED
   0    SRV003        MailboxBackup    Week       NBUSRV      12/04/2010 02:22:32 
 
 Successful jobs during the NAS_Daily_SRV018 backup
 --------------------------------------------------
 STATUS CLIENT        POLICY           SCHED      SERVER      TIME COMPLETED
   0    SRV018        NAS_Daily_SRV018 Full       NBUSRV      12/05/2010 06:19:59 
 
 Successful jobs during the NAS_FileBackup_Citrix backup
 -------------------------------------------------------
 STATUS CLIENT        POLICY           SCHED      SERVER      TIME COMPLETED
   0    SRV015        NAS_FileBackup_C Week       NBUSRV      12/03/2010 19:49:17 
   0    SRV016        NAS_FileBackup_C Week       NBUSRV      12/03/2010 19:48:46 
   0    SRV017        NAS_FileBackup_C Week       NBUSRV      12/03/2010 19:49:29 
 
 Successful jobs during the NAS_FileBackup_DCs backup
 ----------------------------------------------------
 STATUS CLIENT        POLICY           SCHED      SERVER      TIME COMPLETED
   0    SRV007        NAS_FileBackup_D Week       NBUSRV      12/03/2010 20:55:59 
   0    SRV008        NAS_FileBackup_D Week       NBUSRV      12/03/2010 20:33:49 
 
 Successful jobs during the NAS_FileBackup_SQL_WWW backup
 --------------------------------------------------------
 STATUS CLIENT        POLICY           SCHED      SERVER      TIME COMPLETED
   0    SRV009        NAS_FileBackup_S Full       NBUSRV      12/04/2010 00:31:03 
   0    SRV010        NAS_FileBackup_S Full       NBUSRV      12/04/2010 10:21:18 
   0    SRV001        NAS_FileBackup_S Full       NBUSRV      12/03/2010 23:42:43 
   0    SRV013        NAS_FileBackup_S Full       NBUSRV      12/04/2010 00:23:24 
 
 Servers that did not (yet) report any activity
 ----------------------------------------------
 CLIENT         POLICY                   
 SRV022         FileBackup2            
 SRV023         ExchangeBackup           
 SRV024         FileBackup                        
   
 BACKUP STATISTICS
 CLIENT         POLICY                   FILES     SIZE        ELAPSED   
 ------------------------------------------------------------------------
 NBUSRV         CatalogBackup            7.209     7,97 GB     00:11:29
 NBUSRV         FileBackup               32.870    25,60 GB    05:04:51
 SRV001         FileBackup               312.592   118,36 GB   05:25:18
 SRV001         NAS_FileBackup_SQL_WWW   189.777   84,10 GB    03:45:15
 SRV002         Linux_filebackup2        98.381    3,81 GB     00:20:35
 SRV003         ExchangeBackup           9         247,01 GB   09:59:45
 SRV003         FileBackup2              51.021    9,15 GB     00:57:07
 SRV003         MailboxBackup            192.399   5,45 GB     01:22:26
 SRV004         FileBackup               111.210   56,53 GB    06:04:05
 SRV004         NAS_FileBackup_SQL_WWW   61.901    28,16 GB    01:20:48
 SRV005         FileBackup               102.547   185,43 GB   06:44:37
 SRV005         NAS_FileBackup_SQL_WWW   43.621    78,96 GB    03:52:15
 SRV006         PRTG                     48.992    13,36 GB    01:16:57
 SRV007         FileBackup               74.042    27,96 GB    03:01:21
 SRV007         NAS_FileBackup_DCs       37.012    13,92 GB    01:06:18
 SRV008         FileBackup               76.314    14,05 GB    01:47:51
 SRV008         NAS_FileBackup_DCs       38.157    7,00 GB     00:40:53
 SRV009         FileBackup               135.311   182,96 GB   11:30:10
 SRV009         NAS_FileBackup_SQL_WWW   74.393    91,12 GB    02:18:39
 SRV010         FileBackup               118.870   155,18 GB   08:16:04
 SRV010         NAS_FileBackup_SQL_WWW   67.660    57,78 GB    01:27:28
 SRV011         FileBackup               29.968    5,80 GB     00:52:46
 SRV012         FileBackup               84.197    11,41 GB    01:20:36
 SRV013         FileBackup               164.644   259,76 GB   10:42:15
 SRV013         NAS_FileBackup_SQL_WWW   81.413    129,37 GB   04:12:44
 SRV014         FileBackup               34.022    5,87 GB     00:51:19
 SRV015         FileBackup               60.459    16,21 GB    01:10:15
 SRV015         NAS_FileBackup_Citrix    30.224    8,07 GB     00:22:53
 SRV016         FileBackup               73.496    39,25 GB    01:48:33
 SRV016         NAS_FileBackup_Citrix    36.742    19,55 GB    00:26:46
 SRV017         FileBackup               195.726   47,95 GB    02:22:57
 SRV017         NAS_FileBackup_Citrix    97.863    23,88 GB    00:33:53
 SRV018         FileBackup2              5.964.297 505,46 GB   1.06:46:30
 SRV018         NAS_Daily_SRV018         5.921.262 496,49 GB   1.02:24:36
 SRV019         Linux_filebackup         108.119   2,17 GB     00:11:06
 SRV020         Linux_filebackup         127.037   6,28 GB     02:57:13
 SRV021         Linux_filebackup         132.949   7,69 GB     00:18:09
 
 Total Files backedup: 15.016.706
 Total Data backedup: 2,93 TB
 
 Tape Media used during backup
 -----------------------------
 Media ID:          TAPE001
 Last Time Written:       4-12-2010 12:47:33 (1291463253)
 Times Written:     2311 (within this time period)
 Kilobytes:         838688875 (within this time period)
 
 Media ID:          TAPE002
 Last Time Written:       3-12-2010 22:16:36 (1291410996)
 Times Written:     1018 (within this time period)
 Kilobytes:         139560051 (within this time period)
 
 Media ID:          TAPE003
 Last Time Written:       4-12-2010 12:47:29 (1291463249)
 Times Written:     501 (within this time period)
 Kilobytes:         481352638 (within this time period)
 
 Media ID:          CATALOG
 Last Time Written:        6-12-2010 8:02:04 (1291618924)
 Times Written:     6 (within this time period)
 Kilobytes:         8360274 (within this time period)
 
 
 
 Disk Storage Units Status
 -------------------------
 Disk Pool Name      : DISK_POOL1
 Total Capacity (GB) : 300.00
 Free Space (GB)     : 280.76
 Use%                : 6
 Status              : UP
 
 Disk Pool Name      : DISK_POOL2
 Total Capacity (GB) : 5526.45
 Free Space (GB)     : 2460.72
 Use%                : 55
 Status              : UP
 
 
 --------------------------------------------------
 Report finished on NBUSRV, monday 06-12-2010 8:30:51
`

