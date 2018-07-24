---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.1.0
Encoding: ascii
License: gnu gpl
PoshCode ID: 1101
Published Date: 
Archived Date: 2009-05-19t06
---

# geshi powershell syntax - 

## Description

this is the powershell syntax file for geshi that we use on poshcode 1.0

## Comments



## Usage



## TODO



## function

`join-string`

## Code

`#
 #
 <?php
 /*************************************************************************************
  * posh.php
  * ---------------------------------
  * Author: Joel Bennett (Jaykul@HuddledMasses.org)
  * Copyright: (c) 2007 Joel Bennett (http://HuddledMasses.org/)
  * Release Version: 1.1.0
  * Date Started:  2007-06-08
  * Last Modified: 2008-12-27
  *
  * PowerShell language file for GeSHi.
  *
  * The lists of Nouns, Verbs, and Parameters are based on my personal install 
  * with PowerShell Community Extensions installed.  The really bad news is the 
  * fact that aliases are left out almost completely, along with anything from 
  * .Net or COM objects ...
  *
 
 function Join-String { 
   param    ( [string]$separator="', '", [string]$append, [string]$prepend, [string]$prefix="'", [string]$postfix="'")
   begin    { [string[]]$items = @($prepend.split($separator)) }
   process  { $items += $_ }
   end      { $ofs = $separator; $items += @($append.split($separator)); return "$prefix$($items -ne '')$postfix" }
 }
  
  * CHANGES
  * -------
  * 2007-06-08 (0.1.0)
  *  -  First Release
  * 2007-06-09 (1.0.0)
  *  -  Changed to use regular expressions for:
  *     verbs, nouns, and -parameters (was already using them for $variables)
  * 2007-06-10 (1.0.1)
  *  -  Ditched the specific list of parameters in favor of just assuming that
  *     anything that starts with a "-" is a parameter.  Otherwise it won't 
  *     highlight anything that you shorten.
  *  -  Improved the lists by *adding* the "deffinitive" list of verbs from the 
  *     MS CLI spec to my personal verb list.  I didn't remove anything because
  *     ultimately I don't care if it's an official verb if it's a command on my
  *     computer. You should still consider adding your personal verbs too!
  *  -  I improved the 4th Keyword list by exporting my personal list of aliases
  *     and functions ... (and showing the command, so you can generate your own)
  *     and removing the variables entirely (they're already highlighted anyway).
  * 2008-12-27 (1.1.0)
  *  -  Updated for CTP3 with powershell scripts
  *
  * TODO (last updated 2007-06-09)
  * -------------------------
  *  -  I would like to create a script which can dump this whole file out based 
  *     on an individual user's personal particular set of snapins, cmdlets and 
  *     functions.  After all, for YOUR personal site, the cmdlets and functions 
  *     which you have available are the only ones that actually matter :D
  *     HOWEVER: I don't think it's that important, because  
  *
  *************************************************************************************
  *
  *     This file is part of GeSHi.
  *
  *   GeSHi is free software; you can redistribute it and/or modify
  *   it under the terms of the GNU General Public License as published by
  *   the Free Software Foundation; either version 2 of the License, or
  *   (at your option) any later version.
  *
  *   GeSHi is distributed in the hope that it will be useful,
  *   but WITHOUT ANY WARRANTY; without even the implied warranty of
  *   MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
  *   GNU General Public License for more details.
  *
  *   You should have received a copy of the GNU General Public License
  *   along with GeSHi; if not, write to the Free Software
  *   Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA  02111-1307  USA
  *
  ************************************************************************************/
 
 $language_data = array (
   'LANG_NAME' => 'Posh',
   'COMMENT_SINGLE' => array(1 => '#'),
   'COMMENT_MULTI' => array('<#'=>'#>'),
   'CASE_KEYWORDS' => GESHI_CAPS_NO_CHANGE,
   'QUOTEMARKS' => array('"', "'"),
   'HARDQUOTE' => array('@"$','^"@'),          // An optional 2-element array defining the beginning and end of a hard-quoted string
   'HARDESCAPE' => array(),  // Things that must still be escaped inside a hard-quoted string
                 // If HARDQUOTE is defined, HARDESCAPE must be defined
                 // This will not work unless the first character of each element is either in the
                 // QUOTEMARKS array or is the ESCAPE_CHAR
   'ESCAPE_CHAR' => '`',
   'KEYWORDS' => array(
       1 => array( 
 'while', 'until', 'try', 'trap', 'throw', 'switch', 'return', 'process', 'param', 'in', 'if', 'function', 'from', 'foreach', 'for', 'finally', 'filter', 'exit', 'end', 'elseif', 'else', 'dynamicparam', 'do', 'data', 'continue', 'catch', 'break', 'begin'
       ),
       2 => array(
 'xor', 'split', 'replace', 'or', 'notmatch', 'notlike', 'notcontains', 'not', 'ne', 'match', 'lt', 'like', 'le', 'join', 'isplit', 'isnot', 'is', 'ireplace', 'inotmatch', 'inotlike', 'inotcontains', 'ine', 'imatch', 'ilt', 'ilike', 'ile', 'igt', 'ige', 'ieq', 'icontains', 'gt', 'ge', 'f', 'eq', 'csplit', 'creplace', 'contains', 'cnotmatch', 'cnotlike', 'cnotcontains', 'cne', 'cmatch', 'clt', 'clike', 'cle', 'cgt', 'cge', 'ceq', 'ccontains', 'bxor', 'bor', 'bnot', 'band', 'as', 'and'
       ),
       3 => array(
 'xml', 'wmisearcher', 'wmiclass', 'wmi', 'type', 'system', 'switch', 'string', 'single', 'scriptblock', 'runspacefactory', 'runspace', 'regex', 'ref', 'psobject', 'psmoduleinfo', 'pscustomobject', 'powershell', 'microsoft', 'long', 'ipaddress', 'int', 'hashtable', 'float', 'double', 'decimal', 'char', 'byte', 'bool', 'array', 'adsisearcher', 'adsi'
 		),
       4 => array(
 'write', 'wjb', 'which', 'where', 'type', 'tee', 'TabExpansion', 'swmi', 'sv', 'Start', 'spsv', 'spps', 'spjb', 'sp', 'sort', 'sleep', 'sl', 'sign', 'si', 'set', 'select', 'sc', 'sbp', 'say', 'sasv', 'sas', 'sal', 'sajb', 'rwmi', 'rvpa', 'rv', 'rsnp', 'rsn', 'rp', 'rnp', 'rni', 'rmdir', 'rm', 'rjb', 'ri', 'ren', 'rdr', 'rd', 'rcjb', 'rbp', 'RandomLine', 'rand', 'r', 'pwd', 'pushd', 'ps', 'prompt', 'popd', 'oh', 'ogv', 'nv', 'nsn', 'nmo', 'ni', 'ndr', 'nal', 'mv', 'mp', 'move', 'mount', 'more', 'mkdir', 'mi', 'measure', 'md', 'man', 'ls', 'lp', 'kill', 'join', 'iwmi', 'IPSN', 'ipcsv', 'ipal', 'imo', 'ii', 'ihy', 'iex', 'icm', 'history', 'help', 'h', 'gwmi', 'gv', 'gu', 'gsv', 'gsnp', 'gsn', 'group', 'grid', 'gq', 'gps', 'gph', 'gp', 'gmo', 'gm', 'gl', 'gjb', 'gi', 'ghy', 'gdr', 'gcs', 'gcm', 'gci', 'gc', 'gbp', 'gas', 'gal', 'fw', 'ft', 'foreach', 'fl', 'fc', 'EXSN', 'exec', 'ETSN', 'erase', 'EPSN', 'epcsv', 'epal', 'emm', 'edit', 'echo', 'ebp', 'dir', 'diff', 'del', 'dbp', 'cvpa', 'cpp', 'cpi', 'cp', 'copy', 'compare', 'clv', 'cls', 'clp', 'cli', 'clhy', 'clear', 'clc', 'chdir', 'cd\\', 'cd\.\.', 'cd', 'cat', 'asnp', 'ac'
       ),
    ),
    'SYMBOLS' => array(
     '(', ')', '[', ']', '{', '}', "-", "+", "=", '!', '%', '&', '*', '|', '/', '<', '>',
    ),
    'CASE_SENSITIVE' => array(
       GESHI_COMMENTS => true,
       1 => false,
       2 => false,
       3 => false,
       4 => false,
       5 => false,
       6 => false,
    ),
    'STYLES' => array(
       'KEYWORDS' => array(
       ),
       'COMMENTS' => array(
       ),
       'ESCAPE_CHAR' => array(
       ),
       'BRACKETS' => array(
       ),
       'STRINGS' => array(
       ),
       'NUMBERS' => array(
       ),
       'METHODS' => array(
       ),
       'SYMBOLS' => array(
       ),
       'REGEXPS' => array(
       ),
       'SCRIPT' => array(
       )
    ),
    'URLS' => array(
    ),
    'OOLANG' => true,
    'OBJECT_SPLITTERS' => array(
       1 => '.',
       2 => '::',
    ),
    'REGEXPS' => array(
       0 => array (
          GESHI_SEARCH => '((?:Write|Where|Wait|Use|Update|Unregister|Undo|Trace|Test|Tee|Suspend|Stop|Start|Split|Sort|Show|Set|Send|Select|Resume|Restore|Restart|Resolve|Reset|Rename|Remove|Register|Receive|Read|Push|Pop|Out|New|Move|Measure|Limit|Join|Invoke|Import|Group|Get|Format|ForEach|Export|Exit|Enter|Enable|Disconnect|Disable|Debug|Copy|ConvertTo|ConvertFrom|Convert|Connect|Complete|Compare|Clear|Checkpoint|Add)-[a-zA-Z_][a-zA-Z0-9_]*)',
          GESHI_REPLACE => '\\1',
          GESHI_MODIFIERS => 'i',
          GESHI_BEFORE => '',
          GESHI_AFTER => ''
       ),
       1 => array (
          GESHI_SEARCH => '((?:Write|Where|Wait|Use|Update|Unregister|Undo|Trace|Test|Tee|Suspend|Stop|Start|Split|Sort|Show|Set|Send|Select|Resume|Restore|Restart|Resolve|Reset|Rename|Remove|Register|Receive|Read|Push|Pop|Out|New|Move|Measure|Limit|Join|Invoke|Import|Group|Get|Format|ForEach|Export|Exit|Enter|Enable|Disconnect|Disable|Debug|Copy|ConvertTo|ConvertFrom|Convert|Connect|Complete|Compare|Clear|Checkpoint|Add)-)([a-zA-Z_][a-zA-Z0-9_]*)',
          GESHI_REPLACE => '\\2',
          GESHI_MODIFIERS => 'i',
          GESHI_BEFORE => '\\1',
          GESHI_AFTER => ''
       ),      
       2 => array (
          GESHI_SEARCH => ' (-[a-zA-Z_][a-zA-Z0-9_]*)',
          GESHI_REPLACE => '\\1',
          GESHI_MODIFIERS => 'i',
          GESHI_BEFORE => ' ',
          GESHI_AFTER => ''
       ),
       3 => array (
          GESHI_SEARCH => '(\\$[a-zA-Z_][a-zA-Z0-9_]*)',
          GESHI_REPLACE => '\\1',
          GESHI_MODIFIERS => '',
          GESHI_BEFORE => '',
          GESHI_AFTER => ''
       ),
       4 => array (
          GESHI_SEARCH => '(\[[a-z][a-z0-9_\.\]\[]+\])',
          GESHI_REPLACE => '\\1',
          GESHI_MODIFIERS => 'i',
          GESHI_BEFORE => '',
          GESHI_AFTER => ''
       ),
    ),
    'STRICT_MODE_APPLIES' => GESHI_NEVER,
    'SCRIPT_DELIMITERS' => array(
    ),
    'HIGHLIGHT_STRICT_BLOCK' => array(
    )
 );
 
 ?>
`

