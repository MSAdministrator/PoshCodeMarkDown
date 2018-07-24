---
Author: sunnychakraborty
Publisher: 
Copyright: 
Email: 
Version: 1.5.0
Encoding: ascii
License: cc0
PoshCode ID: 3587
Published Date: 2012-08-21t10
Archived Date: 2012-11-27t07
---

# add counterpaths 2 mongo - 

## Description

the script takes about 100 counter paths, and pushes the raw value to a mongo database.

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
 
 .DESCRIPTION
 Similar in vein to Add-EventLogs to MongoDB. http://poshcode.org/3586
 The script takes about 100 counter paths, and pushes the raw value to a mongo database.
 Script can be modified to add CookedValue instead of Raw Value.
 TimeStamps are in UTC -4 (EDT)
 You can watch the events using MongoVue
 http://www.mongovue.com/
 
 Script needs some clean-up to accept input from pipeline for computername, counter names etc.
 .STATUS 
 Alpha - FASTPUBLISH
 
 .TODO
 a) Check if script can accept -continious
 
 #>
 
 $mongoDriverPath = 'C:\Program Files (x86)\MongoDB\CSharpDriver 1.5'
 Add-Type -Path "$($mongoDriverPath)\MongoDB.Bson.dll";
 Add-Type -Path "$($mongoDriverPath)\MongoDB.Driver.dll";
 
 $db = [MongoDB.Driver.MongoDatabase]::Create('mongodb://localhost/eventsX');
 $collection = $db['counters'];
 
 $counters= "memory", "processor", "logicaldisk", "physicaldisk"
 $paths = (get-counter -List $counters).paths 
 
 $samples = (get-counter -Counter $paths).CounterSamples
 
 foreach ($i in $samples) {
 
 [MongoDB.Bson.BsonDocument] $doc = @{
     "_id"= [MongoDB.Bson.ObjectId]::GenerateNewId();
     "Time"= $i.TimeBase;
     "Counters"= [MongoDB.Bson.BsonDocument] [ordered] @{
         "Path"= $i.Path;
         "RawValue"= $i.RawValue;
         "TimeStamp"= ($i.Timestamp).AddMinutes(-240);
         "TimeStamp100nsec"= $i.Timestamp100NSec;
 
 $collection.Insert($doc);
`

