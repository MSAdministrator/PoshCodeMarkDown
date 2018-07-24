---
Author: geoff guynn
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6322
Published Date: 2016-04-25t14
Archived Date: 2016-09-30t12
---

# see you posh cowboy - 

## Description

a silly remake of the see you space cowboy scripts that doesn’t write a file to disk.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function cya($spacecowboy){
   $spacecowboy = "
   IC5kODg4OGI      uICA4ODg4ODg4ODg4I  Dg4ODg4ODg4ODggICA  gICBZO      DhiICA
   gZDg4UCAgLmQ     4ODg4OGIuICA4ODggI  CAgIDg4OA0KZDg4UCA  gWTg4Y      iA4ODg
   gICAgI  CAgIDg   4OCAgI      CAgICA  gICAgI              CAgWTg      4YiBkO
   DhQICB   kODhQI  iAiWTg      4YiA4O  DggICA              gIDg4O      A0KICJ
   ZODg4Y   i4gICA  4ODg4O      Dg4ICA  gIDg4O              Dg4ODg      gICAgI
   CAgICA   gICBZO  Dg4UCA      gICA4O  DggICA              gIDg4O      CA4ODg
   gICAgI   Dg4OA0  KICAgI      CJZODh  iLiA4O              DggICA      gICAgI
   Dg4OCA  gICAgI   CAgICA      gICAgI  CAgODg4ICAgICA4ODg  gICAgIDg4OCA4ODggI
   CAgIDg4OA0KI     CAgICA      gIjg4O  CA4ODggICAgICAgIDg  4OCAgICAgICAgICAgI
   CAgICAgODg       4ICAgI      CA4ODg              gICAgI  Dg4OCA      4ODggI
   CAgIDg4          OA0KWT      g4YiAg              ZDg4UC  A4ODgg      ICAgIC
   AgIDg4           OCAgIC      AgICAg              ICAgIC  AgICAg      ODg4IC
   AgICBZ           ODhiLi      AgZDg4              UCBZOD  hiLiAu      ZDg4UA
   0KICJZ           ODg4OF      AiICA4              ODg4OD  g4ODg4      IDg4OD
   g4ODg4           ODggICAgICAgICAgOD  g4ICAgICAgIlk4ODg4  OFAiIC      AgIlk4
   ODg4OF           AiDQogLmQ4ODg4Yi4g  IDg4ODg4ODhiLiAgIC  AgZDg4      ODggIC
   
   
   5kODg4OGIuICA4O  Dg4ODg4ODg4DQpkODh  wICBZODhiID      g4OCAgIFk4OGIgICBk
   ODg4ODggZDg4UCA  gWTg4YiA4ODgNCiAiW  Tg4OGIuICAgO     Dg4ICAgZDg4UCBkODh
   QIDg4O           CA4ODg      gICAgI  CAgIDg  4ODg4O   DgNCiA             
   gICAiW           Tg4Yi4      gODg4O  Dg4OFA   iIGQ4O  FAgIDg            
   4OCA4O           DggICA      gICAgI  Dg4OA0   KICAgI  CAgIjg            
   4OCA4O           DggICA      gICBkO  DhQICA   gODg4I  Dg4OCA            
   gICA4O           DggODg      4DQpZO  DhiICB   kODhQI  Dg4OCA            
   gICAgZ           Dg4ODg      4ODg4O  DggWTg   4YiAgZ  Dg4UCA4ODgNCiAiWTg
   4ODhQI           iAgODg      4ICAgI  GQ4OFA   gICAgI  Dg4OCAgIlk4ODg4UCI
   gIDg4O           Dg4ODg      4ODgNC  iAuZDg   4ODhiL  iAgIC5            
   kODg4O           DhiLiA      gODg4I  CAgICA   gIDg4O  CA4ODg            
   4ODhiL           iAgICA      uZDg4O  Dg4Yi4   gWTg4Y  iAgIGQ            
   4OFANC           mQ4OFA      gIFk4O  GIgZDg   4UCIgI  lk4OGI            
   gODg4I           CAgbyA      gIDg4O  CA4ODg  gICI4O   GIgIGQ            
   4OFAiICJZODhiIF  k4OGIgZDg4UA0KODg4  ICAgICAgICA4     ODggICAgIDg4OCA4OD
   ggZDg4OGIgODg4I  Dg4ODg4ODhLLiAgODg  4ICAgICA4OD      ggICBZODg4UA0KODg4
   
   ICAgICAgICA4ODggICAgIDg4OCA4ODg4ODg4ODg4ODg4IDg4OCAgIlk4OGIgODg4ICAgICA4
    ODggICAgODg4DQo4ODggICAgODg4IDg4OCAgICAgODg4IDg4ODg4UCBZODg4ODggODg4IC
     AgIDg4OCA4ODggICAgIDg4OCAgICA4ODgNClk4OGIgIGQ4OFAgWTg4Yi4gLmQ4OFAgOD
      g4OFAgICBZODg4OCA4ODggICBkODhQIFk4OGIuIC5kODhQICAgIDg4OA0KICJZODg4
       OFAiICAgIlk4ODg4OFAiICA4ODhQICAgICBZODg4IDg4ODg4ODhQIiAgICJZODg4
                             ODhQIiAgICAgODg4
   " -join "`n"
   
   $colour=("red","yellow","darkyellow","green","cyan","darkcyan","darkmagenta")
   [int]$num=-1
   
   $bytes  = [System.Convert]::FromBase64String($spacecowboy)
   $info = [System.Text.Encoding]::UTF8.GetString($bytes) -split "`n"
   Write-Host ""
   foreach($i in $info){
     [int]$perspective=($num / 3)
     write-host $i -foregroundcolor $colour[$perspective]
     $num++   
   }
     Write-Host ""
 }
 
 clear
 cya($spacecowboy)
`

