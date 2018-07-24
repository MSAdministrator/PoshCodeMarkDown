---
Author: sunnychakraborty
Publisher: 
Copyright: 
Email: 
Version: 1.5
Encoding: ascii
License: cc0
PoshCode ID: 3586
Published Date: 2015-08-20t16
Archived Date: 2015-08-20t01
---

# add eventlogs to mongo - 

## Description

insert event logs to mongodb.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 .NOTES
 AUTHOR: Sunny Chakraborty(sunnyc7@gmail.com)
 WEBSITE: http://tekout.wordpress.com
 CREATED: 8/20/2012
 Requires: 
 a) PowerShell v2 or better
 https://github.com/mongodb/mongo-csharp-driver/downloads
 Tested using 1.5.0.X
 c) Thanks to Justin Dearing for this: 
 https://gist.github.com/854911
 blog post- http://www.justaprogrammer.net/2012/04/19/powershell-3-0-ordered-and-mongodb/
 
 .DESCRIPTION
 Insert event logs in mongodB.
 
 You can watch the events using MongoVue
 http://www.mongovue.com/
 
 #>
 
 $mongoDriverPath = 'C:\Program Files (x86)\MongoDB\CSharpDriver 1.5'
 Add-Type -Path "$($mongoDriverPath)\MongoDB.Bson.dll";
 Add-Type -Path "$($mongoDriverPath)\MongoDB.Driver.dll";
 
 $db = [MongoDB.Driver.MongoDatabase]::Create('mongodb://localhost/eventmonitor');
 $collection = $db['events'];
 
 $events = get-eventlog Application -newest 10
 
 for ($i = 0; $i -le $events.Count-1; $i++) {
 
 
 $a = $events[$i].Index
 $b= $events[$i].EntryType
 $c= $events[$i].InstanceID
 $d= $events[$i].Category
 $e= $events[$i].ReplacementStrings
 $f= $events[$i].Source
 $g= $events[$i].TimeGenerated
 $h= $events[$i].EventID
 $msg = $events[$i].Message
 
 [MongoDB.Bson.BsonDocument] $doc = @{
     "_id"= [MongoDB.Bson.ObjectId]::GenerateNewId();
     "Index"= "$a";
     "EntryType"= "$b";
     "InstanceID"= "$c";
     "Category"= "$d";
     "ReplacementStrings"= "$e";
     "Source"= "$f";
     "TimeGenerated"= "$g";
     "EventID"= "$h";
     "Message"= "$msg";
     };
 
 $collection.Insert($doc);
 }
`

