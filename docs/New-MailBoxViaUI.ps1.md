---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.3
Encoding: ascii
License: cc0
PoshCode ID: 3551
Published Date: 2012-07-28t12
Archived Date: 2012-07-31t01
---

# new-mailboxviaui - 

## Description

a showui function for generating mailboxes with a quick form

## Comments



## Usage



## TODO



## function

`new-mailboxviaui`

## Code

`#
 #
 function New-MailBoxViaUI {
    $MailboxInfo = UniformGrid -ControlName "GetMailboxInfo" -Columns 2 {
       Label "First Name:"
       TextBox -Name FirstName
 
       Label "Last Name:"
       TextBox -Name "LastName"
 
       Label "Mailbox Name:"
       TextBox -Name "Name"
       
       Button -Content "Cancel" -IsCancel -On_Click {
           Get-ParentControl | 
               Close-Control
       }    
       Button "Ok" -IsDefault -On_Click {
           Get-ParentControl | 
               Set-UIValue -passThru | 
               Close-Control
       }
    } -On_Load { 
       $this.Children[1].Focus() 
    } -On_PreviewMouseLeftButtonDown { 
       if($_.Source -notmatch ".*\.(TextBox|Button)") { $ShowUI.ActiveWindow.DragMove() }
    } -Show 
 
    New-Mailbox @MailboxInfo
 }
`

