---
Author: chad miller
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 724
Published Date: 2009-12-09t17
Archived Date: 2016-10-27t12
---

# librarymscs - 

## Description

windows server 2008 r2/powershell 2.0 will include cluster cmdlets, until then this script provides a library functions for working with microsft cluster services (mscs) using the wmi mscluster* class and parsing the output of cluster.exe

## Comments



## Usage



## TODO



## script

`get-clusterlist`

## Code

`#
 #
 
 #######################
 function Get-ClusterList
 {
     $cmd = `cluster.exe /LIST`
 
     for ( $i=3; $i -le ($cmd.length - 1); $i++)
     {
         if ( $cmd[$i] -match '\w+' )
         { $cmd[$i].TrimEnd() }
     }
 
 
 #######################
 function Get-ClusterToNode
 {
     foreach ($cluster in $input) { 
         trap {Write-Error "Cannot connect to $i.";continue} 
         Get-WmiObject -class MSCluster_Node -namespace "root\mscluster" -computername $cluster |
         add-member noteproperty Cluster $cluster -pass | select Cluster, Name 
     }
 
 
 #######################
 function Get-ClusterToVirtual
 {
     foreach ($cluster in $input) { 
 
         $pv = '.*VirtualServerName\s+(?<virtual>\w+$)'
         $pi = '.*InstanceName\s+(?<instance>\w+$)'
         $seen = @()
 
         $cmd = cluster.exe $cluster res /Priv | select-string "VirtualServerName|InstanceName"
 
         for ( $i=0; $i -le ($cmd.length - 1); $i++)
         {
             if ( $cmd[$i] -match $pv )
             {
                 $virtual = $matches.virtual
                 
                 $cmd[$i+1] -match $pi > $null
                 $instance = $matches.instance 
 
                 if (!($seen -contains $virtual))
                 {
                     $seen += $virtual 
                     new-object psobject |
                     add-member -pass NoteProperty Cluster $cluster |
                     add-member -pass NoteProperty Virtual $virtual |
                     add-member -pass NoteProperty Instance $instance
                 }
 
 
             }
         }   
     }
 
 
 #######################
 function Get-ClusterPreferredNode
 {
     param($cluster)
 
     
     
     
     $pg = 'MSCluster_ResourceGroup.Name="(?<group>[^"]+)'
     $pn = 'MSCluster_Node.Name="(?<node>[^"]+)'
 
     get-wmiobject -class MSCluster_ResourceGroupToPreferredNode -namespace "root\mscluster" -computername $cluster |
         select groupcomponent, partcomponent | 
         foreach {   if ($_.GroupComponent -match $pg) {
                                                         add-member -in $_ -membertype noteproperty clustername $cluster
                                                         add-member -in $_ -membertype noteproperty groupname $matches.group
                                                         if ($grp -ne $matches.group)
                                                             { $i = 1; $grp = $matches.group}
                                                         else
                                                             { $i++ }
                                                         add-member -in $_ -membertype noteproperty order $i
 
                                                       } 
                     if ($_.PartComponent  -match $pn) {add-member -in $_ -membertype noteproperty node $matches.node -passthru}
                 } | select clustername, order, groupname, node
 
 
 #######################
 function Get-ClusterGroup
 {
     param($cluster)
 
     #$grpArray = @()
     #$grpArray | foreach { cluster . group "$_" /offline /wait}
 
     $p = '(?<group>^\w+\s?\w*)\s+(?<node>\w+)\s+(?<status>\w+$)' 
     $cmd = `cluster $cluster group`
 
     for ( $i=8; $i -le ($cmd.length - 1); $i++)
     {
         if ( $cmd[$i] -match $p )
         {
             new-object psobject |
             add-member -pass NoteProperty groupname $matches.group.TrimEnd() |
             add-member -pass NoteProperty node $matches.node |
             add-member -pass NoteProperty status $matches.status
         }
     }
 
`

