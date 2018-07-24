---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.2
Encoding: ascii
License: cc0
PoshCode ID: 1750
Published Date: 2011-04-09t14
Archived Date: 2017-02-26t03
---

# log4net.xslt - 

## Description

a better xsl stylesheet for log4net. auto-refreshes, maintains scroll position.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <?xml version="1.0" encoding="ISO-8859-1"?>
 <xsl:transform xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
                xmlns="http://logging.apache.org/log4net/schemas/log4net-events-1.2"
                version="1.0">
    <xsl:output method="html" indent="yes" encoding="UTF-8"/>
    <xsl:template match="events">
       <html><head>
       <!-- refresh every minute (60 seconds)-->
       <meta http-equiv="refresh" content="60" />
       <style>
       th { vertical-align: top; }
       td { padding: 0 6px; border-right: 1px solid black; }
       table.data { width: 100% }
       td.name, td.value { padding: 0px; border: none; }
       td.value { text-align: right }
       </style>
       <!-- You gotta love this: http :// en.hasheminezhad.com/scrollsaver -->
       <script type="text/javascript">
       <xsl:text disable-output-escaping="yes"><![CDATA[(function(){function ls(){var c=document.cookie.split(';');for(var i=0;i<c.length;i++){var p=c[i].split('=');if(p[0]=='scrollPosition'){p=unescape(p[1]).split('/');for(var j=0;j<p.length;j++){var e=p[j].split(',');try{if(e[0]=='window'){window.scrollTo(e[1],e[2]);}else if(e[0]){var o=document.getElementById(e[0]);o.scrollLeft=e[1];o.scrollTop=e[2];}}catch(ex){}}return;}}}function ss(){var s='scrollPosition=';var l,t;if(window.pageXOffset!==undefined){l=window.pageXOffset;t=window.pageYOffset;}else if(document.documentElement&&document.documentElement.scrollLeft!==undefined){l=document.documentElement.scrollLeft;t=document.documentElement.scrollTop;}else{l=document.body.scrollLeft;t=document.body.scrollTop;}if(l||t){s+='window,'+l+','+t+'/';}var es=(document.all)?document.all:document.getElementsByTagName('*');for(var i=0;i<es.length;i++){var e=es[i];if(e.id&&(e.scrollLeft||e.scrollTop)){s+=e.id+','+e.scrollLeft+','+e.scrollTop+'/';}}document.cookie=s+';';}var a,p;if(window.attachEvent){a=window.attachEvent;p='on';}else{a=window.addEventListener;p='';}a(p+'load',function(){ls();if(typeof Sys!='undefined'&&typeof Sys.WebForms!='undefined'){Sys.WebForms.PageRequestManager.getInstance().add_endRequest(ls);Sys.WebForms.PageRequestManager.getInstance().add_beginRequest(ss);}},false);a(p+'unload',ss,false);})();]]></xsl:text>
       </script>
       </head><body id="body">
       <h2><xsl:value-of select="logname"/>.xml Log</h2>
       <table cellspacing="0">
          <tr><th rowspan="2">Logger</th><th rowspan="2">Timestamp</th><th rowspan="2">Level</th><th rowspan="2">Thread</th><th rowspan="2">Message</th><th rowspan="2">NDC</th><th colspan="2">Properties</th><th rowspan="2">Location</th></tr>
          <tr><th>Name</th><th>Value</th></tr>
          <xsl:apply-templates select="event"/>
       </table>
       </body></html>
    </xsl:template>
 
    <xsl:template match="event">
         <tr class="{@level}">
             <td class="logger"><xsl:value-of select="@logger"/></td>
             <td class="timestamp"><xsl:value-of select="@timestamp"/></td>
             <td class="level"><xsl:value-of select="@level"/></td>
             <td class="thread"><xsl:value-of select="@thread"/></td>
             <td class="message"><xsl:apply-templates select="message"/></td>
             <td class="NDC"><xsl:apply-templates select="NDC"/><xsl:text disable-output-escaping="yes"><![CDATA[&nbsp;]]></xsl:text></td>
             <td class="properties" colspan="2"><table class="data"><xsl:apply-templates select="properties/data"/></table></td>
             <td class="locationInfo"><xsl:apply-templates select="locationInfo"/></td>
         </tr>
     </xsl:template>
 
     <xsl:template match="locationInfo">
       <xsl:value-of select="@class"/> <xsl:value-of select="@method"/> <xsl:value-of select="@file"/> <xsl:value-of select="@line"/>
     </xsl:template>
 
     <xsl:template match="data">
         <tr><td class="name"><xsl:value-of select="substring-after(@name,'log4net:')"/><xsl:text disable-output-escaping="yes"><![CDATA[:&nbsp;]]></xsl:text></td><td class="value"><xsl:value-of select="@value"/></td></tr>
     </xsl:template>
 
     <xsl:template match="*">
         <xsl:element name="{name()}" namespace="{namespace-uri()}">
             <xsl:apply-templates select="@*"/>
             <xsl:apply-templates select="*|text()"/>
         </xsl:element>
     </xsl:template>
 
     <xsl:template match="@*">
         <xsl:attribute name="{name()}" namespace="{namespace-uri()}">
             <xsl:value-of select="."/>
         </xsl:attribute>
     </xsl:template>
     
     <xsl:template match="text()">
         <xsl:value-of select="."/>
     </xsl:template>
 </xsl:transform>
`

