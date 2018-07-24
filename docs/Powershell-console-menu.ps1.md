---
Author: h_david_a
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6318
Published Date: 2017-04-24t18
Archived Date: 2017-05-13t16
---

# powershell console menu - 

## Description

simple menu

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $VerbosePreference="continue"
 $FileSeparator=" "
 $menufile="C:\Temp\menu1.txt"
 $keys=@{
     up='w';
     down='s';
     left='a';
     right='d';
     exit='x';
     exec='e'
 }
 
 
 $menu=New-Object -TypeName psobject -Property @{
     'x'=0;
     'y'=0;
     'menu'=''
     }
 $menu|Add-Member -MemberType scriptmethod -Name 'show' -Value {
         cls
         $mm=$this.'menu'
         for ($y=0;$y -lt $this.menu.count; $y++){
             for ($x=0; $x -lt $this.menu[$y].count; $x++){
                 if ($x -eq $this.x -and $y -eq $this.y){
                     Write-Host ("[{0}]" -f $this.menu[$y][$x]) -NoNewline -backgroundcolor green
                     Write-Host " " -NoNewline
                     }
                 else {Write-Host ("{0}{1}" -f $this.menu[$y][$x], " ") -NoNewline}
             }
             Write-Host ""
         }
         write-verbose ("x:{0} $y:{1}" -f $this.x, $this.y)
    }
 $menu|Add-Member -MemberType scriptmethod -Name 'moveup' -Value {
         if ($this.y -gt 0){$this.y-=1}
         else {$this.y= $this.menu.count-1}
         }
 $menu|Add-Member -MemberType scriptmethod -Name 'movedown' -Value {
         if ($this.y -lt $this.menu.count-1){$this.y+=1}
         else {$this.y=0}
         }
 $menu|Add-Member -MemberType scriptmethod -Name 'moveleft' -Value {
         if ($this.x -gt 0){$this.x-=1}
         else {$this.x=$this.menu[$this.y].count-1}
         }
 $menu|Add-Member -MemberType scriptmethod -Name 'moveright' -Value {
         if ($this.x -lt $this.menu[$this.y].count-1){$this.x+=1}
         else {$this.x=0}
         }
 
     
 
 $menu|Add-Member -MemberType scriptmethod -Name 'init' -Value {
         $this.menu=@()
         Get-Content $MENUFILE|%{
         $this.menu+=,@($_.split($FILESEPARATOR))
         }
     
     }
 
 $menu|Add-Member -MemberType scriptmethod -Name 'exec' -Value {
         try{
         }
         catch {}
         
     
     }
 
 
 
 $menu.init()
 #$menu.menu
 while ($key -ne $keys.'exit'){
     $menu.show()
     $key = ([Console]::ReadKey($true)).keychar
     if ($key -eq $keys.up){$menu.moveup()}
     if ($key -eq $keys.down){$menu.movedown()}
     if ($key -eq $keys.left){$menu.moveleft()}
     if ($key -eq $keys.right){$menu.moveright()}
     if ($key -eq $keys.exec){$menu.exec()}
    
     
 }
`

