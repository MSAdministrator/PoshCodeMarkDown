---
Author: james
Publisher: 
Copyright: 
Email: 
Version: 2.0
Encoding: ascii
License: cc0
PoshCode ID: 2510
Published Date: 2011-02-17t07
Archived Date: 2015-11-08t10
---

# powerbot - 

## Description

powerbot is my irc bot written in powershell script using smartirc4net  there’s a bit more to it than this, but this is the basic script, and all you have to do is add your own commands!  of course, you could also add your own additional message handlers and make a chatter-bot or whatever you like. please share your mods back here!

## Comments



## Usage



## TODO



## function

`start-powerbot`

## Code

`#
 #
 ####################################################################################################
 ####################################################################################################
 $null = [Reflection.Assembly]::LoadFrom("$ProfileDir\Libraries\Meebey.SmartIrc4net.dll")
 
 function Start-PowerBot {
   
    if(!$global:irc) { 
       $global:irc = New-Object Meebey.SmartIrc4net.IrcClient
       $irc.Add_OnQueryMessage( {PrivateMessage} )
       $irc.Add_OnChannelMessage( {ChannelMessage} )
    }
    
    $irc.Connect($server, $port)
    $irc.Login($nick, $realname, 0, $nick, $password)
    foreach($channel in $channels) { $irc.RfcJoin( $channel ) }
 }
 
 function Resume-PowerBot {
    while(!$Host.UI.RawUI.KeyAvailable) { $irc.ListenOnce($false) }
 }
 
 function Stop-PowerBot {
    $irc.RfcQuit("If people listened to themselves more often, they would talk less.")
    $irc.Disconnect()
 }
 
 ####################################################################################################
 ####################################################################################################
 ##
 
 function PrivateMessage { 
    $Data = $_.Data
    
    $command, $params = $Data.MessageArray
    if($PowerBotCommands.ContainsKey($command)) {
       &$PowerBotCommands[$command] $params $Data |  Out-String -width (510 - $Data.From.Length - $nick.Length - 3) | % { $_.Trim().Split("`n") | %{ $irc.SendMessage("Message", $Data.Channel, $_.Trim() ) }}
    }
 }
 
 function ChannelMessage {
    $Data = $_.Data
    
    $command, $params = $Data.MessageArray
    if($PowerBotCommands.ContainsKey($command)) {
       &$PowerBotCommands[$command] $params $Data | Out-String -width (510 - $Data.Channel.Length - $nick.Length - 3) | % { $_.Trim().Split("`n") | %{ $irc.SendMessage("Message", $Data.Channel, $_.Trim() ) }}
    }
 }
 
 ####################################################################################################
 ##
 ##
 ##
 $PowerBotCommands=@{}
 
 $PowerBotCommands."Hello" = {Param($Query,$Data)
    "Hello, $($Data.Nick)!"
 }
 
 ##$PowerBotCommands."!Echo" = {Param($Query,$Data)
 ##}
 ##
 
 ##$PowerBotCommands."!Get-Help" = {Param($Query)
 ##}
`

