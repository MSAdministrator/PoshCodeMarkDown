---
Author: steven leise
Publisher: 
Copyright: 
Email: 
Version: 2.0
Encoding: ascii
License: cc0
PoshCode ID: 5975
Published Date: 2016-08-12t14
Archived Date: 2016-05-17t13
---

# help - 

## Description

i need help using the greater than and less than symbol, and it needs to compare two different variables within the code.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #
 #
 
 
 Clear-Host
 
 
 Write-Host "`n`n`n`n`t     W E L C O M E   T O   T H E   G U E S S   M Y"
 Write-Host "`n`n`n`t`t`tN U M B E R   G A M E"
 Write-Host "`n`n`n`t`t`tBy Steven Leise"
 Write-Host "`n`n`t`If you would like to quit at any time enter the letter Q"
 Write-Host "`n`n`n`n`n`n`n`n`n`n Press Enter to continue."
 
 Read-Host
 
 
 
 while ($playGame -ne "No") {
 
 $number = $randomNo.Next(1, 101)
 
 Clear-Host
   while ($status -ne "Stop") {
 
     while ($guess -eq "") {
 
       Write-Host
       $guess = Read-Host " Enter a number between 1 and 100"
 
     }
 
     $noOfGuesses++
 
         Clear-Host
         Write-Host "`n`n"
         Write-Host " Game is now over, thanks for playing Guess My Number!"
         Write-Host "`n`n`n`n`n`n`n`n`n`n`n`n`n`n`n`n"
         Write-Host " Press ENTER to view game stats and quit the game."
         
         
         
         Write-Host "`n Game Statistics"
         Write-Host " ------------------------------------------------------------"
         Write-Host "`n You guessed $noOfGuesses times.`n"
         Write-Host " ------------------------------------------------------------"
         Write-Host "`n`n`n`n`n`n`n`n`n`n`n`n`n`n Press Enter to continue."
 
   Read-Host
 
   Clear-Host
 
 
 break
 
 }
 
 
       Write-Host "`n Sorry. Your guess was too low. You are getting warm. Press Enter to" `
         "guess again."
 
     }
 
       Write-Host "`n Sorry. Your guess was too high. You are getting hot. Press Enter to" `
         "guess again."
 
     }
 
 
       Write-Host "`n Sorry. Your guess was too high. You are getting warmer. Press Enter to" `
         "guess again."
 
     }
 
       Write-Host "`n Sorry. Your guess was too high. You are getting hot. Press Enter to" `
         "guess again."
 
     }
 
 
       Write-Host "`n Congratulations. You guessed my number! Press Enter" `
         "to continue."
 
    }
 
   }
 
   Clear-Host
 
   Write-Host "`n Game Statistics"
   Write-Host " ------------------------------------------------------------"
   Write-Host "`n The secret number was: $number."
   Write-Host "`n You guessed it in $noOfGuesses guesses.`n"
   Write-Host " ------------------------------------------------------------"
   Write-Host "`n`n`n`n`n`n`n`n`n`n`n`n`n`n Press Enter to continue."
 
   Read-Host
 
   Clear-Host
 
 
   while ($reply -eq "") {
 
 
     Write-Host
 
     $reply = Read-Host " Would you like to play again? (Y/N) "
 
      
     if (($reply -ne "Y") -and ($reply -ne "N")) {
 
 
    }
 
   }
 
   if ($reply -eq "Y") {
 
     $number = 0
     $noOfGuesses = 0
     $status = "Play"
     $guess = 0
 
   }
 
 
  }  
 
 }
 
 Clear-Host
`

