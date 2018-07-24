---
Author: joel bennnett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 178
Published Date: 
Archived Date: 2009-11-28t08
---

# start-ircjabberbridge - 

## Description

creates a bridge between (any) muc chatroom and a jabber user — by default it joins the #powershell channel at irc.freenode.net and echos everything said there to you, and everything you say to it … to the chat room.  really quite useless, except as a demonstration.

## Comments



## Usage



## TODO



## function

`start-powerbot`

## Code

`#
 #
 ##########################################################################################
 ##########################################################################################
 
 param (
    $JabberId    =  $(Read-Host "Jabber Login (eg: 'mybot@im.flosoft.biz') ")
    ,$Password   =  $(Read-Host "Jabber Password")
    ,$IRC        =  $(Read-Host "First Jabber conference address (IRC) (eg: powershell%irc.freenode.net@irc.im.flosoft.biz )")
    ,$JabberConf =  $(Read-Host "Second Jabber conference address (eg: powershell@conference.im.flosoft.biz) ")
    ,$ChatNick   =  $(Read-Host "The nickname you want to use")
    ,$IRCPassword=  $(Read-Host "IRC Password")
    ,[string[]]$AtomFeeds = @("http://groups.google.com/group/microsoft.public.windows.powershell/feed/atom_v1_0_topics.xml")
 )
 [Reflection.Assembly]::LoadWithPartialName("System.Web") | Out-Null
 
 
 function Start-PowerBot([switch]$announce) {
    $global:LastNewsCheck = [DateTime]::Now
    $global:feedReader = new-object Xml.XmlDocument
    $global:PoshXmppClient = PoshXmpp\New-Client $JabberId $Password http://im.flosoft.biz:5280/http-poll/
    PoshXmpp\Connect-Chat $IRC $ChatNick
    PoshXmpp\Connect-Chat $JabberConf $ChatNick
 
    $global:PoshXmppClient.SendMyPresence()
    
    if($announce) {
       PoshXmpp\Send-Message $IRC "Starting Mirror to $('{0}@{1}' -f $JabberConf.Split(@('%','@'),3))"
       PoshXmpp\Send-Message $JabberConf "Starting Mirror to $('{0}@{1}' -f $IRC.Split(@('%','@'),3))"
    }
    if($IRCPassword) {
 	  Send-PrivMsg "nickserv" "id $IRCPassword"
    }
   
 }
 
 function Send-PrivMsg($to,$msg) {
     PoshXmpp\Send-Message $IRC "/msg $to :$msg"
 }
 
 function Process-Command($Chat, $Message) {
    $split = $message.Body.Split(" |;")
    $from = $Message.From.Bare.ToLower()
    switch -regex ( $split[0] ) {
       "!list" {
          Write-Host "!LIST COMMAND. Send users of [$Chat] to [$($Message.From.Bare)]" -fore Magenta
          $users = @($PoshXmppClient.ChatManager.GetUsers( $Chat ).Values)
          PoshXmpp\Send-Message $from  ("There are $($users.Count) on $($Chat):")
          PoshXmpp\Send-Message $from  [string].join(', ',$users)
       }
       "!gh|!get-help|!man" {
          $help = get-help $split[1] | Select Name,Synopsis,Syntax
          if($?) {
             if($help -is [array]) {
                PoshXmpp\Send-Message $from ("You're going to need to be more specific, I know all about $((($help | % { $_.Name })[0..($help.Length-2)] -join ', ') + ' and even ' + $help[-1].Name)")
             } else {
                PoshXmpp\Send-Message $from $help.Synopsis;
                PoshXmpp\Send-Message $from ($help.Syntax | Out-String -w 1000).Split("`n",4,[StringSplitOptions]::RemoveEmptyEntries)[1]
             }
          } else {
             PoshXmpp\Send-Message $from "I couldn't find the help file for that, sorry.  Jaykul probably didn't get the cmdlet installed right."
          }
       }
    }
 }
 
 
 $IrcMaxLen = 510 - ("PRIVMSG :".Length + $IRC.Length + $JabberId.split('@')[0].Length)
 
 function Get-Feeds([string[]]$JIDs,[string[]]$AtomFeeds) {
    Write-Verbose "Checking feeds..."
    foreach($feed in $AtomFeeds) {
       $feedReader.Load($feed)
       for($i = $feedReader.feed.entry.count - 1; $i -ge 0; $i--) {
          $e = $feedReader.feed.entry[$i]
          if([datetime]$e.updated -gt $global:LastNewsCheck) {
             foreach($jid in $JIDs) {
                $template = "{0} (Posted at {1:hh:mm} by {2}) {{0}} :: {3}" -f
 
                $len = [math]::Min($IrcMaxLen,($template.Length + $msg.Length)) - ($template.Length +3)
                PoshXmpp\Send-Message $jid $($template -f $msg.SubString(0,$len))
             }
             [Threading.Thread]::Sleep( 500 )
          }
       }
    }
    $global:LastNewsCheck = [DateTime]::Now
 }
 
 [regex]$anagram = "^Unscramble ... (.*)$"
 
 function Bridge {
 	foreach( $msg in (PoshXmpp\Receive-Message -All) )
 	{
       $Chat = $Null;
       if( $msg.Type -eq "Error" )
       { 
          Write-Error $msg.Error 
       }
       elseif( $msg.From.Resource -ne $ChatNick ) 
       {
          switch($msg.From.Bare) {
             $IRC        { $Chat = $JabberConf }
             $JabberConf { $Chat = $IRC;       }
             default      { Write-Debug $msg }
          }
       }
       if($Chat) {
          switch($msg) {
             { ![String]::IsNullOrEmpty($_.Body) -and ($_.Body[0] -eq '!') }{ 
          	   PoshXmpp\Send-Message $Chat ("<{0}> {1}" -f $_.From.Resource, $_.Body)
          	   Process-Command $Chat $_ 
          	} 
             { ($_.From.Resource -eq "GeoBot") -and $_.Body.StartsWith("Unscramble ... ") }{
                Write-Verbose "KILL ANAGRAM! $($anagram.Match($_.Body).Groups[1].value)"
                $answers = Solve-Anagram $($anagram.Match($_.Body).Groups[1].value)
                foreach($ans in $answers) {
                   Write-Verbose "ANAGRAM: $($_.From.Bare) $ans"
                   PoshXmpp\Send-Message "$($_.From.Bare.ToLower())" "$ans (PowerShell Scripting FTW!)"
                }
             }
             { ![String]::IsNullOrEmpty($_.Subject) }{
                $_.To = $Chat
                $PoshXmppClient.Send($_)
             }
             { $_.From.Resource }{
                PoshXmpp\Send-Message $Chat ("<{0}> {1}" -f $_.From.Resource, $_.Body)
             }
          }
       }
    }
 }
 
 
 function Start-Bridge {
    "PRESS ANY KEY TO STOP"
    while(!$Host.UI.RawUI.KeyAvailable) {
       $Host.UI.RawUI.WindowTitle = "PowerBot"
       $global:PoshXmppClient.SendMyPresence()
       Bridge
       Get-Feeds @($IRC,$JabberConf) $AtomFeeds
       $counter = 0
       while(!$Host.UI.RawUI.KeyAvailable -and ($counter++ -lt 1800)) {
          Bridge
          [Threading.Thread]::Sleep( 100 )
       }
    }
 }
 
 function Stop-PowerBot {
    PoshXmpp\Disconnect-Chat $IRC $ChatNick
    PoshXmpp\Disconnect-Chat $JabberConf $ChatNick
    $global:PoshXmppClient.Close();
 }
 
 ##
 ##
 function Solve-Anagram($anagram) {
    ((Post-HTTP "http://www.easypeasy.com/anagrams/results.php" "name=$anagram").Split("`n") |
     select-string "res 1" ) -replace ".*res 1.*value=""\s*([^""]*)\s*"".*",'$1'
 }
 
 function Post-HTTP($url,$bytes) {
    $request = [System.Net.WebRequest]::Create($url)
    $request.ContentType = "application/x-www-form-urlencoded"
    $request.ContentLength = $bytes.Length
    $request.Method = "POST"
    $rq = new-object IO.StreamWriter $request.GetRequestStream()
    $rq.Write($bytes)#,0,$bytes.Length)
    $rq.Flush()
    $rq.Close()
    $response = $request.GetResponse()
    $reader = new-object IO.StreamReader $response.GetResponseStream(),[Text.Encoding]::UTF8
    return $reader.ReadToEnd()
 }
`

