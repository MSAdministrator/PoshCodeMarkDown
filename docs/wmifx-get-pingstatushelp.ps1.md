---
Author: andreas
Publisher: 
Copyright: 
Email: 
Version: 1.1.0.9
Encoding: utf-8
License: cc0
PoshCode ID: 3541
Published Date: 2012-07-26t03
Archived Date: 2012-07-31t01
---

# wmifx get-pingstatushelp - 

## Description

content of the get-pingstatus-help.xml file uploaded for kirk for troubleshooting reasons

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <helpItems Schema="maml">
 	<!-- v 1.1.0.9 -->
 	<command:command xmlns:maml="http://schemas.microsoft.com/maml/2004/10" xmlns:command="http://schemas.microsoft.com/maml/dev/command/2004/10" xmlns:dev="http://schemas.microsoft.com/maml/dev/2004/10">
 		<command:details>
 			<!-- NAME -->
 			<command:name>Get-PingStatus</command:name>
 			<maml:description>
 				<!-- SYNOPSIS -->
 				<maml:para>Gets instances of Win32_PingStatus Windows Management Instrumentation (WMI) class. Die Klasse "Win32_PingStatus" enthält Werte, die vom Standardpingbefehl zurückgegeben werden.</maml:para>
 			</maml:description>
 			<maml:copyright>
 				<maml:para />
 			</maml:copyright>
 			<command:verb>Get</command:verb>
 			<command:noun>PingStatus</command:noun>
 			<dev:version />
 		</command:details>
 		<maml:description>
 			<!-- DESCRIPTION -->
 			<maml:para>The Get-PingStatus command gets instances of Win32_PingStatus WMI class.
 
 Die Klasse "Win32_PingStatus" enthält Werte, die vom Standardpingbefehl zurückgegeben werden. Weitere Informationen über Ping erhalten Sie unter RFC 791.
 
 The ComputerName parameter can always be used to specify a remote computer.
 
 The Get-PingStatus command does not use the Windows PowerShell remoting infrastructure to perform remote operations. You can use the ComputerName parameter of the Get-PingStatus command even if your computer does not meet the requirements for Windows PowerShell remoting and even if your computer is not configured for remoting in Windows PowerShell.</maml:para>
 		</maml:description>
 		<!-- SYNTAX -->
 		<command:syntax>
 			<command:syntaxItem>
 				<maml:name>Get-PingStatus</maml:name>
 				<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="1">
 					<maml:name>Address</maml:name>
 					<maml:description>
 						<maml:para>Specifies Address as a filter.
 
 Die Eigenschaft "Address" enthält den Wert der angeforderten Adresse. Dies kann der Hostname ('wxyz1234') oder die IP-Adresse ('193.128.177.124') sein.
 
 Supports exact matches or wildcards. When multiple values are provided, any of those values will result in a match.</maml:para>
 					</maml:description>
 					<command:parameterValue required="false" variableLength="false">String[]</command:parameterValue>
 				</command:parameter>
 				<command:parameter required="false" variableLength="true" globbing="false" pipelineInput="false" position="2">
 					<maml:name>Property</maml:name>
 					<maml:description>
 						<maml:para></maml:para>
 					</maml:description>
 					<command:parameterValue required="false" variableLength="true">String[]</command:parameterValue>
 				</command:parameter>
 				<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="2">
 					<maml:name>BufferSize</maml:name>
 					<maml:description>
 						<maml:para>Specifies BufferSize as a filter.
 
 Die Eigenschaft "BufferSize" zeigt die mit dem Pingbefehl gesendete Puffergröße an. Der Standardwert beträgt 32.
 
 Supports exact matches or wildcards. When multiple values are provided, any of those values will result in a match.</maml:para>
 					</maml:description>
 					<command:parameterValue required="false" variableLength="false">UInt32[]</command:parameterValue>
 				</command:parameter>
 				<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="3">
 					<maml:name>NoFragmentation</maml:name>
 					<maml:description>
 						<maml:para>Specifies NoFragmentation as a filter.
 
 Die Eigenschaft "NoFragmentation" bestimmt, ob gesendete Pakete fragmentiert werden. Der Standardwert ist False: nicht fragmentieren.
 
 Supports exact matches or wildcards. When multiple values are provided, any of those values will result in a match.</maml:para>
 					</maml:description>
 					<command:parameterValue required="false" variableLength="false">Boolean[]</command:parameterValue>
 				</command:parameter>
 				<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="4">
 					<maml:name>RecordRoute</maml:name>
 					<maml:description>
 						<maml:para>Specifies RecordRoute as a filter.
 
 Die Eigenschaft "RecordRoute" bestimmt, wie viele Hops während der Paketweiterleitung aufgezeichnet werden. Der Standardwert ist 0.
 
 Supports exact matches or wildcards. When multiple values are provided, any of those values will result in a match.</maml:para>
 					</maml:description>
 					<command:parameterValue required="false" variableLength="false">UInt32[]</command:parameterValue>
 				</command:parameter>
 				<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="5">
 					<maml:name>ResolveAddressNames</maml:name>
 					<maml:description>
 						<maml:para>Specifies ResolveAddressNames as a filter.
 
 Die Eigenschaft "ResolveAddressesNames" gibt an, ob der Befehl Adressnamen von Ausgabeadresswerten auflöst. Der Standardwert ist False - keine Auflösung.
 
 Supports exact matches or wildcards. When multiple values are provided, any of those values will result in a match.</maml:para>
 					</maml:description>
 					<command:parameterValue required="false" variableLength="false">Boolean[]</command:parameterValue>
 				</command:parameter>
 				<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="6">
 					<maml:name>SourceRoute</maml:name>
 					<maml:description>
 						<maml:para>Specifies SourceRoute as a filter.
 
 Die Eigenschaft "SourceRoute" enthält eine durch Trennzeichen getrennte Liste gültiger Quellrouten.
 
 Supports exact matches or wildcards. When multiple values are provided, any of those values will result in a match.</maml:para>
 					</maml:description>
 					<command:parameterValue required="false" variableLength="false">String[]</command:parameterValue>
 				</command:parameter>
 				<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="7">
 					<maml:name>SourceRouteType</maml:name>
 					<maml:description>
 						<maml:para>Specifies SourceRouteType as a filter.
 
 Die Eigenschaft "SourceRouteType" gibt den Typ der in der Hostliste zu verwendeten Quellrouteoption an, der in der Eigenschaft "SourceRoute" angegeben ist. Falls ein Wert außerhalb von "ValueMap" angegeben wird, wird der Wert 0 verwendet. Der Standardwert ist null.
 
 Supports exact matches or wildcards. When multiple values are provided, any of those values will result in a match.</maml:para>
 					</maml:description>
 					<command:parameterValue required="false" variableLength="false">UInt32[]</command:parameterValue>
 				</command:parameter>
 				<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="8">
 					<maml:name>Timeout</maml:name>
 					<maml:description>
 						<maml:para>Specifies Timeout as a filter.
 
 Die Eigenschaft "Timeout" zeigt den Zeitüberschreitungswert in Millisekunden an. Falls keine Antwort in dieser Zeit empfangen wurde, wird davon ausgegangen, dass keine Antwort gesendet wurde. Der Standardwert beträgt 4000 Millisekunden.
 
 Supports exact matches or wildcards. When multiple values are provided, any of those values will result in a match.</maml:para>
 					</maml:description>
 					<command:parameterValue required="false" variableLength="false">UInt32[]</command:parameterValue>
 				</command:parameter>
 				<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="9">
 					<maml:name>TimestampRoute</maml:name>
 					<maml:description>
 						<maml:para>Specifies TimestampRoute as a filter.
 
 Die Eigenschaft "TimestampRoute" gibt an, wie viele Hops mit den Zeitstempelinformationen aufgezeichnet werden, während das Paket unterwegs ist. Ein Zeitstempel ist die Anzahl der Millisekunden, die seit dem Mitternachts-UT verstrichen sind. Falls die Dauer nicht in Millisekunden oder nicht im Verhältnis zum Mitternachts-UT angezeigt werden kann, kann irgendeine Dauer in den Zeitstempel eingefügt werden, solange das hochwertige Bit des Zeitstempelfelds auf eins gesetzt ist, was auf einen nicht standardmäßigen Wert verweist. Der Standardwert ist null.
 
 Supports exact matches or wildcards. When multiple values are provided, any of those values will result in a match.</maml:para>
 					</maml:description>
 					<command:parameterValue required="false" variableLength="false">UInt32[]</command:parameterValue>
 				</command:parameter>
 				<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="10">
 					<maml:name>TimeToLive</maml:name>
 					<maml:description>
 						<maml:para>Specifies TimeToLive as a filter.
 
 Die Eigenschaft "TimeToLive" zeigt die Gültigkeitsdauer des Pingpakets in Sekunden an. Dies ist der höhere und kein absoluter Grenzwert, da alle Router diesen Wert um eins herunterstufen MÜSSEN und Hops zwischen Routern selten soviel Zeit benötigen. Falls dieser Wert auf 0 gesetzt wird, wird das Paket vom Router verworfen. Der Standardwert beträgt 80 Sekunden.
 
 Supports exact matches or wildcards. When multiple values are provided, any of those values will result in a match.</maml:para>
 					</maml:description>
 					<command:parameterValue required="false" variableLength="false">UInt32[]</command:parameterValue>
 				</command:parameter>
 				<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="11">
 					<maml:name>TypeofService</maml:name>
 					<maml:description>
 						<maml:para>Specifies TypeofService as a filter.
 
 Die Eigenschaft "TypeOfService" verweist auf abstrakte Parameter des gewünschten QoS (Quality of Service). Diese Parameter sollten die Auswahl der tatsächlichen Dienstparameter beim Übertragen eines Datagramms über ein bestimmtes Netzwerk erleichtern. Der Standardwert ist 0. 
 
 
 
 
 Bits 0-2:  Vorrang. 
 
 
 
 
 Bit  	3:  0 = Normale Verzögerung,  		1 = Kurze Verzögerung. 
 
 
 
 
 Bits   4:  0 = Normaler Durchsatz, 1 = Hoher Durchsatz. 
 
 
 
 
 Bits   5:  0 = Normale Verlässlichkeit, 1 = Hohe Verlässlichkeit. 
 
 
 
 
 Bit  6-7:  Reserviert. 
 
 
 
 
 Vorrang 
 
 
 
 
 111 - Netzwerksteuerung 
 
 
 
 
 110 - Interne Netzwerksteuerung 
 
 
 
 
 101 - CRITIC/ECP 
 
 
 
 
 100 - Flashaußerkraftsetzung 
 
 
 
 
 011 - Flash 
 
 
 
 
 010 - Sofort 
 
 
 
 
 001 - Priorität 
 
 
 
 
 000 - Routine  
 
 
 
 
  
 
 
 
 
 Eine detaillierte Beschreibung der verschiedenen Diensttypen erhalten Sie in RFC 791, Seite 12.
 
 Supports exact matches or wildcards. When multiple values are provided, any of those values will result in a match.</maml:para>
 					</maml:description>
 					<command:parameterValue required="false" variableLength="false">UInt32[]</command:parameterValue>
 				</command:parameter>
 				<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="named">
 					<maml:name>SearchProperty</maml:name>
 					<maml:description>
 						<maml:para>Specifies individual properties and specific values to use as a filter. Supports exact matches or wildcards. When multiple properties are provided, only objects matching all conditions will be returned. When multiple values are provided for a single property, any of those values will result in a match.</maml:para>
 					</maml:description>
 					<command:parameterValue required="false" variableLength="false">Hashtable</command:parameterValue>
 				</command:parameter>
 				<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="named">
 					<maml:name>Credential</maml:name>
 					<maml:description>
 						<maml:para></maml:para>
 					</maml:description>
 					<command:parameterValue required="false" variableLength="false">PSCredential</command:parameterValue>
 				</command:parameter>
 				<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="named">
 					<maml:name>ThrottleLimit</maml:name>
 					<maml:description>
 						<maml:para></maml:para>
 					</maml:description>
 					<command:parameterValue required="false" variableLength="false">Int32</command:parameterValue>
 				</command:parameter>
 				<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="named">
 					<maml:name>EnableAllPrivileges</maml:name>
 					<maml:description>
 						<maml:para></maml:para>
 					</maml:description>
 					<command:parameterValue required="false" variableLength="false">SwitchParameter</command:parameterValue>
 				</command:parameter>
 				<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="named">
 					<maml:name>Authority</maml:name>
 					<maml:description>
 						<maml:para></maml:para>
 					</maml:description>
 					<command:parameterValue required="false" variableLength="false">String</command:parameterValue>
 				</command:parameter>
 				<command:parameter required="false" variableLength="true" globbing="false" pipelineInput="false" position="named">
 					<maml:name>ComputerName</maml:name>
 					<maml:description>
 						<maml:para></maml:para>
 					</maml:description>
 					<command:parameterValue required="false" variableLength="true">String[]</command:parameterValue>
 				</command:parameter>
 				<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="named">
 					<maml:name>Filter</maml:name>
 					<maml:description>
 						<maml:para></maml:para>
 					</maml:description>
 					<command:parameterValue required="false" variableLength="false">String</command:parameterValue>
 				</command:parameter>
 				<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="named">
 					<maml:name>Impersonation</maml:name>
 					<maml:description>
 						<maml:para></maml:para>
 					</maml:description>
 					<command:parameterValue required="false" variableLength="false">ImpersonationLevel</command:parameterValue>
 				</command:parameter>
 				<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="named">
 					<maml:name>JobName</maml:name>
 					<maml:description>
 						<maml:para>Specifies a friendly name for the background job. By default, jobs are named "Job&lt;n&gt;", where &lt;n&gt; is an ordinal number.
 
 If you use the JobName parameter in a command, the command is run as a job, and Get-PingStatus returns a job object, even if you do not include the AsJob parameter in the command.
 
 For more information about Windows PowerShell background jobs, see about_Jobs (http://go.microsoft.com/fwlink/?LinkID=113251).</maml:para>
 					</maml:description>
 					<command:parameterValue required="false" variableLength="false">String</command:parameterValue>
 				</command:parameter>
 				<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="named">
 					<maml:name>AsJob</maml:name>
 					<maml:description>
 						<maml:para></maml:para>
 					</maml:description>
 					<command:parameterValue required="false" variableLength="false">SwitchParameter</command:parameterValue>
 				</command:parameter>
 			</command:syntaxItem>
 		</command:syntax>
 		<!-- PARAMETERS -->
 		<command:parameters>
 			<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="5">
 				<maml:name>ResolveAddressNames</maml:name>
 				<maml:description>
 					<maml:para>Specifies ResolveAddressNames as a filter.
 
 Die Eigenschaft "ResolveAddressesNames" gibt an, ob der Befehl Adressnamen von Ausgabeadresswerten auflöst. Der Standardwert ist False - keine Auflösung.
 
 Supports exact matches or wildcards. When multiple values are provided, any of those values will result in a match.</maml:para>
 				</maml:description>
 				<command:parameterValue required="false" variableLength="false">Boolean[]</command:parameterValue>
 				<dev:type>
 					<maml:name>Boolean[]</maml:name>
 					<maml:uri />
 				</dev:type>
 				<dev:defaultValue />
 			</command:parameter>
 			<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="6">
 				<maml:name>SourceRoute</maml:name>
 				<maml:description>
 					<maml:para>Specifies SourceRoute as a filter.
 
 Die Eigenschaft "SourceRoute" enthält eine durch Trennzeichen getrennte Liste gültiger Quellrouten.
 
 Supports exact matches or wildcards. When multiple values are provided, any of those values will result in a match.</maml:para>
 				</maml:description>
 				<command:parameterValue required="false" variableLength="false">String[]</command:parameterValue>
 				<dev:type>
 					<maml:name>String[]</maml:name>
 					<maml:uri />
 				</dev:type>
 				<dev:defaultValue />
 			</command:parameter>
 			<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="4">
 				<maml:name>RecordRoute</maml:name>
 				<maml:description>
 					<maml:para>Specifies RecordRoute as a filter.
 
 Die Eigenschaft "RecordRoute" bestimmt, wie viele Hops während der Paketweiterleitung aufgezeichnet werden. Der Standardwert ist 0.
 
 Supports exact matches or wildcards. When multiple values are provided, any of those values will result in a match.</maml:para>
 				</maml:description>
 				<command:parameterValue required="false" variableLength="false">UInt32[]</command:parameterValue>
 				<dev:type>
 					<maml:name>UInt32[]</maml:name>
 					<maml:uri />
 				</dev:type>
 				<dev:defaultValue />
 			</command:parameter>
 			<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="2">
 				<maml:name>BufferSize</maml:name>
 				<maml:description>
 					<maml:para>Specifies BufferSize as a filter.
 
 Die Eigenschaft "BufferSize" zeigt die mit dem Pingbefehl gesendete Puffergröße an. Der Standardwert beträgt 32.
 
 Supports exact matches or wildcards. When multiple values are provided, any of those values will result in a match.</maml:para>
 				</maml:description>
 				<command:parameterValue required="false" variableLength="false">UInt32[]</command:parameterValue>
 				<dev:type>
 					<maml:name>UInt32[]</maml:name>
 					<maml:uri />
 				</dev:type>
 				<dev:defaultValue />
 			</command:parameter>
 			<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="3">
 				<maml:name>NoFragmentation</maml:name>
 				<maml:description>
 					<maml:para>Specifies NoFragmentation as a filter.
 
 Die Eigenschaft "NoFragmentation" bestimmt, ob gesendete Pakete fragmentiert werden. Der Standardwert ist False: nicht fragmentieren.
 
 Supports exact matches or wildcards. When multiple values are provided, any of those values will result in a match.</maml:para>
 				</maml:description>
 				<command:parameterValue required="false" variableLength="false">Boolean[]</command:parameterValue>
 				<dev:type>
 					<maml:name>Boolean[]</maml:name>
 					<maml:uri />
 				</dev:type>
 				<dev:defaultValue />
 			</command:parameter>
 			<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="7">
 				<maml:name>SourceRouteType</maml:name>
 				<maml:description>
 					<maml:para>Specifies SourceRouteType as a filter.
 
 Die Eigenschaft "SourceRouteType" gibt den Typ der in der Hostliste zu verwendeten Quellrouteoption an, der in der Eigenschaft "SourceRoute" angegeben ist. Falls ein Wert außerhalb von "ValueMap" angegeben wird, wird der Wert 0 verwendet. Der Standardwert ist null.
 
 Supports exact matches or wildcards. When multiple values are provided, any of those values will result in a match.</maml:para>
 				</maml:description>
 				<command:parameterValue required="false" variableLength="false">UInt32[]</command:parameterValue>
 				<dev:type>
 					<maml:name>UInt32[]</maml:name>
 					<maml:uri />
 				</dev:type>
 				<dev:defaultValue />
 			</command:parameter>
 			<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="11">
 				<maml:name>TypeofService</maml:name>
 				<maml:description>
 					<maml:para>Specifies TypeofService as a filter.
 
 Die Eigenschaft "TypeOfService" verweist auf abstrakte Parameter des gewünschten QoS (Quality of Service). Diese Parameter sollten die Auswahl der tatsächlichen Dienstparameter beim Übertragen eines Datagramms über ein bestimmtes Netzwerk erleichtern. Der Standardwert ist 0. 
 
 
 
 
 Bits 0-2:  Vorrang. 
 
 
 
 
 Bit  	3:  0 = Normale Verzögerung,  		1 = Kurze Verzögerung. 
 
 
 
 
 Bits   4:  0 = Normaler Durchsatz, 1 = Hoher Durchsatz. 
 
 
 
 
 Bits   5:  0 = Normale Verlässlichkeit, 1 = Hohe Verlässlichkeit. 
 
 
 
 
 Bit  6-7:  Reserviert. 
 
 
 
 
 Vorrang 
 
 
 
 
 111 - Netzwerksteuerung 
 
 
 
 
 110 - Interne Netzwerksteuerung 
 
 
 
 
 101 - CRITIC/ECP 
 
 
 
 
 100 - Flashaußerkraftsetzung 
 
 
 
 
 011 - Flash 
 
 
 
 
 010 - Sofort 
 
 
 
 
 001 - Priorität 
 
 
 
 
 000 - Routine  
 
 
 
 
  
 
 
 
 
 Eine detaillierte Beschreibung der verschiedenen Diensttypen erhalten Sie in RFC 791, Seite 12.
 
 Supports exact matches or wildcards. When multiple values are provided, any of those values will result in a match.</maml:para>
 				</maml:description>
 				<command:parameterValue required="false" variableLength="false">UInt32[]</command:parameterValue>
 				<dev:type>
 					<maml:name>UInt32[]</maml:name>
 					<maml:uri />
 				</dev:type>
 				<dev:defaultValue />
 			</command:parameter>
 			<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="named">
 				<maml:name>SearchProperty</maml:name>
 				<maml:description>
 					<maml:para>Specifies individual properties and specific values to use as a filter. Supports exact matches or wildcards. When multiple properties are provided, only objects matching all conditions will be returned. When multiple values are provided for a single property, any of those values will result in a match.</maml:para>
 				</maml:description>
 				<command:parameterValue required="false" variableLength="false">Hashtable</command:parameterValue>
 				<dev:type>
 					<maml:name>Hashtable</maml:name>
 					<maml:uri />
 				</dev:type>
 				<dev:defaultValue />
 			</command:parameter>
 			<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="10">
 				<maml:name>TimeToLive</maml:name>
 				<maml:description>
 					<maml:para>Specifies TimeToLive as a filter.
 
 Die Eigenschaft "TimeToLive" zeigt die Gültigkeitsdauer des Pingpakets in Sekunden an. Dies ist der höhere und kein absoluter Grenzwert, da alle Router diesen Wert um eins herunterstufen MÜSSEN und Hops zwischen Routern selten soviel Zeit benötigen. Falls dieser Wert auf 0 gesetzt wird, wird das Paket vom Router verworfen. Der Standardwert beträgt 80 Sekunden.
 
 Supports exact matches or wildcards. When multiple values are provided, any of those values will result in a match.</maml:para>
 				</maml:description>
 				<command:parameterValue required="false" variableLength="false">UInt32[]</command:parameterValue>
 				<dev:type>
 					<maml:name>UInt32[]</maml:name>
 					<maml:uri />
 				</dev:type>
 				<dev:defaultValue />
 			</command:parameter>
 			<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="8">
 				<maml:name>Timeout</maml:name>
 				<maml:description>
 					<maml:para>Specifies Timeout as a filter.
 
 Die Eigenschaft "Timeout" zeigt den Zeitüberschreitungswert in Millisekunden an. Falls keine Antwort in dieser Zeit empfangen wurde, wird davon ausgegangen, dass keine Antwort gesendet wurde. Der Standardwert beträgt 4000 Millisekunden.
 
 Supports exact matches or wildcards. When multiple values are provided, any of those values will result in a match.</maml:para>
 				</maml:description>
 				<command:parameterValue required="false" variableLength="false">UInt32[]</command:parameterValue>
 				<dev:type>
 					<maml:name>UInt32[]</maml:name>
 					<maml:uri />
 				</dev:type>
 				<dev:defaultValue />
 			</command:parameter>
 			<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="9">
 				<maml:name>TimestampRoute</maml:name>
 				<maml:description>
 					<maml:para>Specifies TimestampRoute as a filter.
 
 Die Eigenschaft "TimestampRoute" gibt an, wie viele Hops mit den Zeitstempelinformationen aufgezeichnet werden, während das Paket unterwegs ist. Ein Zeitstempel ist die Anzahl der Millisekunden, die seit dem Mitternachts-UT verstrichen sind. Falls die Dauer nicht in Millisekunden oder nicht im Verhältnis zum Mitternachts-UT angezeigt werden kann, kann irgendeine Dauer in den Zeitstempel eingefügt werden, solange das hochwertige Bit des Zeitstempelfelds auf eins gesetzt ist, was auf einen nicht standardmäßigen Wert verweist. Der Standardwert ist null.
 
 Supports exact matches or wildcards. When multiple values are provided, any of those values will result in a match.</maml:para>
 				</maml:description>
 				<command:parameterValue required="false" variableLength="false">UInt32[]</command:parameterValue>
 				<dev:type>
 					<maml:name>UInt32[]</maml:name>
 					<maml:uri />
 				</dev:type>
 				<dev:defaultValue />
 			</command:parameter>
 			<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="named">
 				<maml:name>Impersonation</maml:name>
 				<maml:description>
 					<maml:para>TODO: Add a parameter description</maml:para>
 				</maml:description>
 				<command:parameterValue required="false" variableLength="false">ImpersonationLevel</command:parameterValue>
 				<dev:type>
 					<maml:name>ImpersonationLevel</maml:name>
 					<maml:uri />
 				</dev:type>
 				<dev:defaultValue />
 			</command:parameter>
 			<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="named">
 				<maml:name>EnableAllPrivileges</maml:name>
 				<maml:description>
 					<maml:para>TODO: Add a parameter description</maml:para>
 				</maml:description>
 				<command:parameterValue required="false" variableLength="false">SwitchParameter</command:parameterValue>
 				<dev:type>
 					<maml:name>SwitchParameter</maml:name>
 					<maml:uri />
 				</dev:type>
 				<dev:defaultValue />
 			</command:parameter>
 			<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="named">
 				<maml:name>AsJob</maml:name>
 				<maml:description>
 					<maml:para>TODO: Add a parameter description</maml:para>
 				</maml:description>
 				<command:parameterValue required="false" variableLength="false">SwitchParameter</command:parameterValue>
 				<dev:type>
 					<maml:name>SwitchParameter</maml:name>
 					<maml:uri />
 				</dev:type>
 				<dev:defaultValue />
 			</command:parameter>
 			<command:parameter required="false" variableLength="true" globbing="false" pipelineInput="false" position="named">
 				<maml:name>Property</maml:name>
 				<maml:description>
 					<maml:para>Specifies the property or set of properties to retrieve from the Win32_PingStatus objects.</maml:para>
 				</maml:description>
 				<command:parameterValue required="false" variableLength="true">String[]</command:parameterValue>
 				<dev:type>
 					<maml:name>String[]</maml:name>
 					<maml:uri />
 				</dev:type>
 				<dev:defaultValue />
 			</command:parameter>
 			<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="named">
 				<maml:name>Filter</maml:name>
 				<maml:description>
 					<maml:para>TODO: Add a parameter description</maml:para>
 				</maml:description>
 				<command:parameterValue required="false" variableLength="false">String</command:parameterValue>
 				<dev:type>
 					<maml:name>String</maml:name>
 					<maml:uri />
 				</dev:type>
 				<dev:defaultValue />
 			</command:parameter>
 			<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="named">
 				<maml:name>Authority</maml:name>
 				<maml:description>
 					<maml:para>TODO: Add a parameter description</maml:para>
 				</maml:description>
 				<command:parameterValue required="false" variableLength="false">String</command:parameterValue>
 				<dev:type>
 					<maml:name>String</maml:name>
 					<maml:uri />
 				</dev:type>
 				<dev:defaultValue />
 			</command:parameter>
 			<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="named">
 				<maml:name>JobName</maml:name>
 				<maml:description>
 					<maml:para>Specifies a friendly name for the background job. By default, jobs are named "Job&lt;n&gt;", where &lt;n&gt; is an ordinal number.
 
 If you use the JobName parameter in a command, the command is run as a job, and Get-PingStatus returns a job object, even if you do not include the AsJob parameter in the command.
 
 For more information about Windows PowerShell background jobs, see about_Jobs (http://go.microsoft.com/fwlink/?LinkID=113251).</maml:para>
 				</maml:description>
 				<command:parameterValue required="false" variableLength="false">String</command:parameterValue>
 				<dev:type>
 					<maml:name>String</maml:name>
 					<maml:uri />
 				</dev:type>
 				<dev:defaultValue />
 			</command:parameter>
 			<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="1">
 				<maml:name>Address</maml:name>
 				<maml:description>
 					<maml:para>Specifies Address as a filter.
 
 Die Eigenschaft "Address" enthält den Wert der angeforderten Adresse. Dies kann der Hostname ('wxyz1234') oder die IP-Adresse ('193.128.177.124') sein.
 
 Supports exact matches or wildcards. When multiple values are provided, any of those values will result in a match.</maml:para>
 				</maml:description>
 				<command:parameterValue required="false" variableLength="false">String[]</command:parameterValue>
 				<dev:type>
 					<maml:name>String[]</maml:name>
 					<maml:uri />
 				</dev:type>
 				<dev:defaultValue />
 			</command:parameter>
 			<command:parameter required="false" variableLength="true" globbing="false" pipelineInput="false" position="named">
 				<maml:name>ComputerName</maml:name>
 				<maml:description>
 					<maml:para>TODO: Add a parameter description</maml:para>
 				</maml:description>
 				<command:parameterValue required="false" variableLength="true">String[]</command:parameterValue>
 				<dev:type>
 					<maml:name>String[]</maml:name>
 					<maml:uri />
 				</dev:type>
 				<dev:defaultValue />
 			</command:parameter>
 			<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="named">
 				<maml:name>Credential</maml:name>
 				<maml:description>
 					<maml:para>TODO: Add a parameter description</maml:para>
 				</maml:description>
 				<command:parameterValue required="false" variableLength="false">PSCredential</command:parameterValue>
 				<dev:type>
 					<maml:name>PSCredential</maml:name>
 					<maml:uri />
 				</dev:type>
 				<dev:defaultValue />
 			</command:parameter>
 			<command:parameter required="false" variableLength="false" globbing="false" pipelineInput="false" position="named">
 				<maml:name>ThrottleLimit</maml:name>
 				<maml:description>
 					<maml:para>TODO: Add a parameter description</maml:para>
 				</maml:description>
 				<command:parameterValue required="false" variableLength="false">Int32</command:parameterValue>
 				<dev:type>
 					<maml:name>Int32</maml:name>
 					<maml:uri />
 				</dev:type>
 				<dev:defaultValue />
 			</command:parameter>
 		</command:parameters>
 		<!-- INPUTS -->
 		<command:inputTypes>
 			<command:inputType>
 				<dev:type>
 					<maml:name>None</maml:name>
 					<maml:uri></maml:uri>
 					<maml:description></maml:description>
 				</dev:type>
 				<maml:description>
 					<maml:para></maml:para>
 				</maml:description>
 			</command:inputType>
 		</command:inputTypes>
 		<!-- OUTPUTS -->
 		<command:returnValues>
 			<command:returnValue>
 				<dev:type>
 					<maml:uri></maml:uri>
 					<maml:description></maml:description>
 				</dev:type>
 				<maml:description>
 					<maml:para></maml:para>
 				</maml:description>
 			</command:returnValue>
 		</command:returnValues>
 		<command:terminatingErrors />
 		<command:nonTerminatingErrors />
 		<!-- NOTES -->
 		<maml:alertSet>
 			<maml:title />
 			<maml:alert>
 				<maml:para></maml:para>
 			</maml:alert>
 		</maml:alertSet>
 		<!-- EXAMPLES -->
 		<command:examples>
 			<command:example>
 				<maml:title>-------------------------- EXAMPLE 1 --------------------------</maml:title>
 				<maml:introduction>
 					<maml:para>PS C:\&gt;</maml:para>
 				</maml:introduction>
 				<dev:code>Get-PingStatus</dev:code>
 				<dev:remarks>
 					<maml:para>Description</maml:para>
 					<maml:para>-----------</maml:para>
 					<maml:para>This command displays information about Win32_PingStatus objects on a computer.</maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 				</dev:remarks>
 				<command:commandLines>
 					<command:commandLine>
 						<command:commandText />
 					</command:commandLine>
 				</command:commandLines>
 			</command:example>
 			<command:example>
 				<maml:title>-------------------------- EXAMPLE 2 --------------------------</maml:title>
 				<maml:introduction>
 					<maml:para>PS C:\&gt;</maml:para>
 				</maml:introduction>
 				<dev:code>Get-PingStatus -ComputerName 127.0.0.1</dev:code>
 				<dev:remarks>
 					<maml:para>Description</maml:para>
 					<maml:para>-----------</maml:para>
 					<maml:para>This command displays information about Win32_PingStatus objects on the remote computer. It identifies the computer to use by specifying the Internet Protocol (IP) address 127.0.0.1. You can change this IP address to any other valid IP address on your network so that you can display information about the services on that remote computer. By default, the account you are running under must be a member of the local administrators group on the remote computer that you specify.</maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 				</dev:remarks>
 				<command:commandLines>
 					<command:commandLine>
 						<command:commandText />
 					</command:commandLine>
 				</command:commandLines>
 			</command:example>
 			<command:example>
 				<maml:title>-------------------------- EXAMPLE 3 --------------------------</maml:title>
 				<maml:introduction>
 					<maml:para>PS C:\&gt;</maml:para>
 				</maml:introduction>
 				<dev:code>Get-PingStatus | Format-List *</dev:code>
 				<dev:remarks>
 					<maml:para>Description</maml:para>
 					<maml:para>-----------</maml:para>
 					<maml:para>This command displays Win32_PingStatus object information. It displays all the properties of the Win32_PingStatus WMI class, not just the properties that are specified in the Types.ps1xml configuration file.</maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 				</dev:remarks>
 				<command:commandLines>
 					<command:commandLine>
 						<command:commandText />
 					</command:commandLine>
 				</command:commandLines>
 			</command:example>
 			<command:example>
 				<maml:title>-------------------------- EXAMPLE 4 --------------------------</maml:title>
 				<maml:introduction>
 					<maml:para>PS C:\&gt;</maml:para>
 				</maml:introduction>
 				<dev:code>Get-PingStatus -Credential FABRIKAM\Administrator -ComputerName Fabrikam</dev:code>
 				<dev:remarks>
 					<maml:para>Description</maml:para>
 					<maml:para>-----------</maml:para>
 					<maml:para>This command displays Win32_PingStatus object information on a computer named Fabrikam. It specifies a user account name by using the Credential parameter, which causes a dialog box to be displayed in which you enter the corresponding password.</maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 				</dev:remarks>
 				<command:commandLines>
 					<command:commandLine>
 						<command:commandText />
 					</command:commandLine>
 				</command:commandLines>
 			</command:example>
 			<command:example>
 				<maml:title>-------------------------- EXAMPLE 5 --------------------------</maml:title>
 				<maml:introduction>
 					<maml:para>PS C:\&gt;</maml:para>
 				</maml:introduction>
 				<dev:code>Get-PingStatus -Address c*</dev:code>
 				<dev:remarks>
 					<maml:para>Description</maml:para>
 					<maml:para>-----------</maml:para>
 					<maml:para>This command displays information about any Win32_PingStatus objects on a computer with a Address that starts with 'C'.</maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 				</dev:remarks>
 				<command:commandLines>
 					<command:commandLine>
 						<command:commandText />
 					</command:commandLine>
 				</command:commandLines>
 			</command:example>
 			<command:example>
 				<maml:title>-------------------------- EXAMPLE 6 --------------------------</maml:title>
 				<maml:introduction>
 					<maml:para>PS C:\&gt;</maml:para>
 				</maml:introduction>
 				<dev:code>Get-PingStatus -BufferSize 23</dev:code>
 				<dev:remarks>
 					<maml:para>Description</maml:para>
 					<maml:para>-----------</maml:para>
 					<maml:para>This command displays information about any Win32_PingStatus objects on a computer with a BufferSize value of 23.</maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 				</dev:remarks>
 				<command:commandLines>
 					<command:commandLine>
 						<command:commandText />
 					</command:commandLine>
 				</command:commandLines>
 			</command:example>
 			<command:example>
 				<maml:title>-------------------------- EXAMPLE 7 --------------------------</maml:title>
 				<maml:introduction>
 					<maml:para>PS C:\&gt;</maml:para>
 				</maml:introduction>
 				<dev:code>Get-PingStatus -NoFragmentation e*</dev:code>
 				<dev:remarks>
 					<maml:para>Description</maml:para>
 					<maml:para>-----------</maml:para>
 					<maml:para>This command displays information about any Win32_PingStatus objects on a computer with a NoFragmentation that starts with 'E'.</maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 				</dev:remarks>
 				<command:commandLines>
 					<command:commandLine>
 						<command:commandText />
 					</command:commandLine>
 				</command:commandLines>
 			</command:example>
 			<command:example>
 				<maml:title>-------------------------- EXAMPLE 8 --------------------------</maml:title>
 				<maml:introduction>
 					<maml:para>PS C:\&gt;</maml:para>
 				</maml:introduction>
 				<dev:code>Get-PingStatus -RecordRoute 76</dev:code>
 				<dev:remarks>
 					<maml:para>Description</maml:para>
 					<maml:para>-----------</maml:para>
 					<maml:para>This command displays information about any Win32_PingStatus objects on a computer with a RecordRoute value of 76.</maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 				</dev:remarks>
 				<command:commandLines>
 					<command:commandLine>
 						<command:commandText />
 					</command:commandLine>
 				</command:commandLines>
 			</command:example>
 			<command:example>
 				<maml:title>-------------------------- EXAMPLE 9 --------------------------</maml:title>
 				<maml:introduction>
 					<maml:para>PS C:\&gt;</maml:para>
 				</maml:introduction>
 				<dev:code>Get-PingStatus -ResolveAddressNames e*</dev:code>
 				<dev:remarks>
 					<maml:para>Description</maml:para>
 					<maml:para>-----------</maml:para>
 					<maml:para>This command displays information about any Win32_PingStatus objects on a computer with a ResolveAddressNames that starts with 'E'.</maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 				</dev:remarks>
 				<command:commandLines>
 					<command:commandLine>
 						<command:commandText />
 					</command:commandLine>
 				</command:commandLines>
 			</command:example>
 			<command:example>
 				<maml:title>-------------------------- EXAMPLE 10 --------------------------</maml:title>
 				<maml:introduction>
 					<maml:para>PS C:\&gt;</maml:para>
 				</maml:introduction>
 				<dev:code>Get-PingStatus -SourceRoute h*</dev:code>
 				<dev:remarks>
 					<maml:para>Description</maml:para>
 					<maml:para>-----------</maml:para>
 					<maml:para>This command displays information about any Win32_PingStatus objects on a computer with a SourceRoute that starts with 'H'.</maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 				</dev:remarks>
 				<command:commandLines>
 					<command:commandLine>
 						<command:commandText />
 					</command:commandLine>
 				</command:commandLines>
 			</command:example>
 			<command:example>
 				<maml:title>-------------------------- EXAMPLE 11 --------------------------</maml:title>
 				<maml:introduction>
 					<maml:para>PS C:\&gt;</maml:para>
 				</maml:introduction>
 				<dev:code>Get-PingStatus -SourceRouteType 50</dev:code>
 				<dev:remarks>
 					<maml:para>Description</maml:para>
 					<maml:para>-----------</maml:para>
 					<maml:para>This command displays information about any Win32_PingStatus objects on a computer with a SourceRouteType value of 50.</maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 				</dev:remarks>
 				<command:commandLines>
 					<command:commandLine>
 						<command:commandText />
 					</command:commandLine>
 				</command:commandLines>
 			</command:example>
 			<command:example>
 				<maml:title>-------------------------- EXAMPLE 12 --------------------------</maml:title>
 				<maml:introduction>
 					<maml:para>PS C:\&gt;</maml:para>
 				</maml:introduction>
 				<dev:code>Get-PingStatus -Timeout 67</dev:code>
 				<dev:remarks>
 					<maml:para>Description</maml:para>
 					<maml:para>-----------</maml:para>
 					<maml:para>This command displays information about any Win32_PingStatus objects on a computer with a Timeout value of 67.</maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 				</dev:remarks>
 				<command:commandLines>
 					<command:commandLine>
 						<command:commandText />
 					</command:commandLine>
 				</command:commandLines>
 			</command:example>
 			<command:example>
 				<maml:title>-------------------------- EXAMPLE 13 --------------------------</maml:title>
 				<maml:introduction>
 					<maml:para>PS C:\&gt;</maml:para>
 				</maml:introduction>
 				<dev:code>Get-PingStatus -TimestampRoute 35</dev:code>
 				<dev:remarks>
 					<maml:para>Description</maml:para>
 					<maml:para>-----------</maml:para>
 					<maml:para>This command displays information about any Win32_PingStatus objects on a computer with a TimestampRoute value of 35.</maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 				</dev:remarks>
 				<command:commandLines>
 					<command:commandLine>
 						<command:commandText />
 					</command:commandLine>
 				</command:commandLines>
 			</command:example>
 			<command:example>
 				<maml:title>-------------------------- EXAMPLE 14 --------------------------</maml:title>
 				<maml:introduction>
 					<maml:para>PS C:\&gt;</maml:para>
 				</maml:introduction>
 				<dev:code>Get-PingStatus -TimeToLive 73</dev:code>
 				<dev:remarks>
 					<maml:para>Description</maml:para>
 					<maml:para>-----------</maml:para>
 					<maml:para>This command displays information about any Win32_PingStatus objects on a computer with a TimeToLive value of 73.</maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 				</dev:remarks>
 				<command:commandLines>
 					<command:commandLine>
 						<command:commandText />
 					</command:commandLine>
 				</command:commandLines>
 			</command:example>
 			<command:example>
 				<maml:title>-------------------------- EXAMPLE 15 --------------------------</maml:title>
 				<maml:introduction>
 					<maml:para>PS C:\&gt;</maml:para>
 				</maml:introduction>
 				<dev:code>Get-PingStatus -TypeofService 59</dev:code>
 				<dev:remarks>
 					<maml:para>Description</maml:para>
 					<maml:para>-----------</maml:para>
 					<maml:para>This command displays information about any Win32_PingStatus objects on a computer with a TypeofService value of 59.</maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 					<maml:para></maml:para>
 				</dev:remarks>
 				<command:commandLines>
 					<command:commandLine>
 						<command:commandText />
 					</command:commandLine>
 				</command:commandLines>
 			</command:example>
 		</command:examples>
 		<!-- RELATED LINKS -->
 		<maml:relatedLinks>
 			<maml:navigationLink>
 				<maml:linkText>Get-WmiObject</maml:linkText>
 				<maml:uri />
 			</maml:navigationLink>
 			<maml:navigationLink>
 				<maml:linkText>Invoke-WmiMethod</maml:linkText>
 				<maml:uri />
 			</maml:navigationLink>
 			<maml:navigationLink>
 				<maml:linkText>Remove-WmiObject</maml:linkText>
 				<maml:uri />
 			</maml:navigationLink>
 			<maml:navigationLink>
 				<maml:linkText>Set-WmiInstance</maml:linkText>
 				<maml:uri />
 			</maml:navigationLink>
 		</maml:relatedLinks>
 	</command:command>
 </helpItems>
`

