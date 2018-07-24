---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 892
Published Date: 
Archived Date: 2009-02-25t08
---

# shift operators(correct) - 

## Description

the missing shift operators corrected so they shift in the right direction.

## Comments



## Usage



## TODO



## class

`shift-left`

## Code

`#
 #
 Add-Type @"
 public class Shift {
    public static int   Right(int x,   int count) { return x >> count; }
    public static uint  Right(uint x,  int count) { return x >> count; }
    public static long  Right(long x,  int count) { return x >> count; }
    public static ulong Right(ulong x, int count) { return x >> count; }
    public static int    Left(int x,   int count) { return x << count; }
    public static uint   Left(uint x,  int count) { return x << count; }
    public static long   Left(long x,  int count) { return x << count; }
    public static ulong  Left(ulong x, int count) { return x << count; }
 }                    
 "@
 
 
 
 #.Example 
 #.Example 
 function Shift-Left {
 PARAM( $x=1, $y )
 BEGIN {
    if($y) {
       [Shift]::Left( $x, $y )
    }
 }
 PROCESS {
    if($_){
       [Shift]::Left($_, $x)
    }
 }
 }
 
 
 #.Example 
 #.Example 
 function Shift-Right {
 PARAM( $x=1, $y )
 BEGIN {
    if($y) {
       [Shift]::Right( $x, $y )
    }
 }
 PROCESS {
    if($_){
       [Shift]::Right($_, $x)
    }
 }
 }
`

