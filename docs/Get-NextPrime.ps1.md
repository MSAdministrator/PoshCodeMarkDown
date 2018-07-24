---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2338
Published Date: 
Archived Date: 2010-11-02t18
---

# get-nextprime - 

## Description

calculate prime numbers …

## Comments



## Usage



## TODO



## function

`get-nextprime`

## Code

`#
 #
 $primes = 2,3,5 #,7,11,13,17,19,23
 $primeIndex = 0
 function Get-NextPrime {
    [CmdletBinding(DefaultParameterSetName="KnownPrime")]
    param(
       [Parameter(Position=0,ParameterSetName="KnownPrime")]
       $knownPrime = $(if($primeIndex -lt $primes.Count){ $primes[$primeIndex] } else { $primes[-1] } )
    , 
       [Parameter(ParameterSetName="FromTwo")]
       [switch]$reset
    )
    if($reset) { 
       $global:primeIndex = -1 
    } elseif($PSBoundParameters.ContainsKey("KnownPrime")){
       $global:primeIndex = [array]::BinarySearch($primes,$knownPrime)
       if($primeIndex -lt 0) { 
          Write-Verbose "We don't have a prime like $knownPrime"
          $primeIndex = (-1 * $primeIndex) -1
       } else {
          Write-Verbose "Looking for the first prime LARGER than $($primes[$primeIndex])"
          $global:primeIndex += 1
       }
    }
    
    Write-Verbose "Looking for a prime gt $knownPrime at index $primeIndex"
    if($primeIndex -lt $primes.Count) {
       $primes[$primeIndex]
       $global:primeIndex += 1
       return 
    }
    
    $p = $primes[-1]
    while($p = $p+2){
       if(!$(
          foreach($i in $primes){ if($p % $i -eq 0) { $i } }
       )){
          $global:primes += $p
          $global:primeIndex += 1
          if($p -gt $knownPrime) {
             return $p
          }
       }
    }
 }
`

