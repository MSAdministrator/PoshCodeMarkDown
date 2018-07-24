---
Author: mark nelson
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3731
Published Date: 2012-10-31t06
Archived Date: 2012-11-04t00
---

# product key - 

## Description

ironpython script for retrieving ms products. there is how to get windows key in following example.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 from System import Array, Char, Console
 from System.Collections import ArrayList
 from Microsoft.Win32 import Registry
 
 def DecodeProductKey(digitalProductID):
    map = ("BCDFGHJKMPQRTVWXY2346789").ToCharArray()
    key = Array.CreateInstance(Char, 29)
    raw = ArrayList()
 
    i = 52
    while i < 67:
       raw.Add(digitalProductID[i])
       i += 1
 
    i = 28
    while i >= 0:
       if (i + 1) % 6 == 0:
          key[i] = '-'
       else:
          k = 0
          j = 14
          while j >= 0:
             k = (k * 256) ^ raw[j]
             raw[j] = (k / 24)
             k %= 24
             key[i] = map[k]
             j -= 1
       i -= 1
 
    return key
 
 reg = Registry.LocalMachine.OpenSubKey("SOFTWARE\Microsoft\Windows NT\CurrentVersion")
 val = reg.GetValue("DigitalProductId")
 Console.WriteLine(DecodeProductKey(val))
 
 Hmm, Mark stole my code at http://poshcode.org/3545
`

