---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1743
Published Date: 2011-04-06t22
Archived Date: 2016-03-23t13
---

# default.config - 

## Description

an example config for log4net

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 <!-- An example log4net config (Save as default.config) -->
 <log4net>
    <appender name="ColoredConsoleAppender" type="log4net.Appender.ColoredConsoleAppender">
        <mapping>
            <level value="ERROR" />
            <foreColor value="Red, HighIntensity" />
            <backColor value="White, HighIntensity" />
        </mapping>
        <mapping>
            <level value="DEBUG" />
            <foreColor value="Green, HighIntensity" />
        </mapping>
        <mapping>
            <level value="INFO" />
            <foreColor value="Cyan, HighIntensity" />
        </mapping>
        <mapping>
            <level value="WARN" />
            <foreColor value="Yellow, HighIntensity" />
        </mapping>
            <layout type="log4net.Layout.PatternLayout">
            <conversionPattern value="%date [%thread] %-5level %logger [%property{NDC}] - %message%newline" />
        </layout>
    </appender>
 
    <appender name="RollingFile" type="log4net.Appender.RollingFileAppender">
        <file value="example.log" />
        <appendToFile value="true" />
        <maximumFileSize value="100KB" />
        <maxSizeRollBackups value="2" />
 
        <layout type="log4net.Layout.PatternLayout">
            <conversionPattern value="%level %thread %logger - %message%newline" />
        </layout>
    </appender>
     
     <root>
         <level value="DEBUG" />
         <appender-ref ref="ColoredConsoleAppender" />
         <appender-ref ref="RollingFile" />
     </root>
 </log4net>
`

