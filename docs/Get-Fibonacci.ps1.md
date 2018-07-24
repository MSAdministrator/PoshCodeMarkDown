---
Author: dvspupop
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4146
Published Date: 2013-05-03t23
Archived Date: 2013-05-07t07
---

# get-fibonacci - 

## Description

fast (using text) and easy to get large fibonacci numbers. the 5000th fib takes like 15 secs for my computer. wonder how many loops that is? alot!

## Comments



## Usage



## TODO



## function

`add-strings`

## Code

`#
 #
 function Add-Strings{
     <#
     .SYNOPSIS
     Adds two strings together.
 
     .DESCRIPTION
     Lines the strings up and adds each number separately giving you the sum of the numbers. 
     
     .EXAMPLE
     Add-Strings -Num1 10 -Num2 90
 
     100
 
     Returns a string of the sum of the 2 numbers.
 
     .NOTES
 
     .LINK
     http://www.wikihow.com/Add-Large-Numbers
     #>
     [cmdletbinding()]
     param(
         [Parameter(Mandatory=$true)]
         [Alias('Num1','NumOne')]
         [string]$Left,
         
         [Parameter(Mandatory=$true)]
         [Alias('Num2','NumTwo')]
         [string]$Right
     )
 
     if($Left -match '[a-zA-Z]'){
         Write-Warning 'Left variable is invalid; contains a-z or A-Z'    
         return 0
     }
     if($Right -match '[a-zA-Z]'){
         Write-Warning 'Right variable is invalid; contains a-z or A-Z'    
         return 0
     }
     
 
     $len1 = $Left.length
     $len2 = $Right.length
 
     if($len1-gt$len2){
         for($i=$len2;$i-lt$len1;$i++){
             $Right = '0' + $Right
         }
         $len2 = $len1
     }
     else{
         for($i=$len1;$i-lt$len2;$i++){
             $Left = '0' + $Left
         }
         $len1 = $len2
     }
 
     $out=''
     $carry=0
     for($i=$len1-1;$i-ge0;$i--){
         $sum = [int][string]$Left[$i]+[int][string]$Right[$i] + $carry
         if($sum-ge10){
             $carry=1
             $sum-=10
         }
         else{
             $carry=0
         }
         $out = "$sum$out"
     }
     if($carry){
         $out = "1$out"
     }
     $len = $out.Length
     for($i=0; $i-lt$len;$i++){
         if($out[$i]-eq"0"){
             $out = $out.Substring($i+1)
             $i--
         }
         else{
             break
         }
     }
     Write-Output $out
 }
 
 
 function Get-Fibonacci{
     <#
     .SYNOPSIS
     Returns the requested ordinal Fibonacci number.
 
     .DESCRIPTION
     Using basic arithmetic a human would use to add large numbers, the script is 
     able to go beyond what a basic int32 number can handel to return large Fibonacci numbers.
     
     .EXAMPLE
     Get-Fibonacci -Place 99
 
     218922995834555169026
 
     Returns a string of the 99th Fibonacci number.
 
     .NOTES
     Largest Fibonacci number will be 4,000 digits which I believe is the max length for a string.
 
     .LINK
     http://www.wikihow.com/Add-Large-Numbers
     #>
 
     [cmdletbinding()]
     param(
         [ValidateRange(0,1000000)]
         [int]$Place=10
     )
 
     if($Place -eq 1){
         return '1'
     }
 
     $next = '1'
     $curn = '0'
 
     for($i=0;$i-le($Place-1);$i++){
         Write-Verbose $curn
         $tmpnext = Add-Strings -Num1 $curn -Num2 $next
         $curn = $next
         $next = $tmpnext
     }
     Write-Output $curn
 }
 
 Get-Fibonacci $args[0] #-Verbose
 
`

