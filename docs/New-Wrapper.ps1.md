---
Author: rybat
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2730
Published Date: 2012-06-12t22
Archived Date: 2016-02-20t08
---

# new-wrapper - 

## Description

a script that makes an encrypted powershell script.

## Comments



## Usage



## TODO



## function

`new-wrapper`

## Code

`#
 #
 function New-Wrapper {
 <#
     .Synopsis
         Encrypt a secret script to run from a new ps1 wrapper script.
     
     .Description
         Encrypt a secret script to run from a new ps1 wrapper script.
 
         A block diagram of how password is protected. http://bit.ly/jE1N7n
         The original template used http://bit.ly/lFE2IO if required.
         
         Important - Flatten Original Script first:
         Original script must be prepared because when it is decrypted, it's converted into a big "one liner"....so,
         -Place a semicolon ; at the end of each line.
         -Remove unnecessary white space. 
        
         The password is embedded in the new script's file-name for convenience, so rename before you run it for the first time.
         If savePass option is used, it will not prompt for a password again unless script is altered, renamed, moved or 
         executed from a different host like the ISE.
 
         Optionally: 
         Script can be locked to a specific user (most secure mode)
         Forced to be run from UNC - to thwart debugging
         Save encoded password in registry - so you only need to enter it once. 
         Obfuscate wrapper script code - easy to read variable names are replaced with hard to read names. 
         
         No, it is not 100% secure, but with the option to use perUser it is secure as the user's account,
         and if password is not even saved, then it is as secure as AES.
         
         Basically if anything, when the password is saved, the script is signed. i.e. If a single character in the 
         script is altered, then the script will cease to run and will prompt for the original password again. 
         
         MD5 hash is used instead of SHA1 because it conveniently makes a compatible 128 bit key length key for the 128 bit AES.
         The known MD5 vulnerabilities are not an issue in this implementation. 
            
                                    
     .parameter inputScript
         Script you want to encrypt and wrap.
                                           
     .parameter password
         Password to encrypt and decrypt script
                                           
     .parameter perUser
         Each user that wants to run script will have to have to enter the password.
         Used with savePass option, the user's identity is hashed with the password as extra protection. 
         Only the owner of this user account can possibly reverse engineer the password making it more secure.
                        
     .parameter forceUnC
         Script will only run from UNC, thus thwarting debugger from extracting decrypted script and password. 
         Also if script is saved on a secure network share, separare from password then that could be desirable. 
                                      
     .parameter savePass
         XOR Hash of hardware finger print and password is saved in registry - to decode password, script must XOR again with hashes of 
         the script's path, script's contents, system serial number, PS host that the script is running under and the 
         identity of user (if specified).
                               
     .parameter test
         Quickly allows you to test script runs from encrypted block before wrapping. If original script has not been 
         properly prepared, ie. flattened as per instructions in description above, then it will fail. 
        
     .parameter newTemplate 
         Use this switch to build a new wrapper template. 
         Input Script is output to Hex so you can use this block to replace the $template variable. 
         (Note, if copied from console, remove carriage returns)
     
     .parameter obfuscate 
         Easy to read variable names are replaced with hard to read names. Note Security through obscurity is bad so 
         if the script has to be really secure then use peruser mode or don't save the password. 
         The wrapper script is very hard to understand when obfuscated but an experienced attacker with time could 
         unravel and then attempt to manually build the hardware hash.                       
       
   
    .Example
        New-Wrapper "C:\scripts\secret.ps1"
        
     Description
     -----------
         Encrypts secret.ps1 and places it in a PowerShell wrapper script using:
         Default password "Password1"
         Current folder and default file name
           
                       
          
    .Example
        New-Wrapper -inputScript "C:\scripts\secret.ps1" -password 43rH3489Dv -savePass -perUser -obfuscate
        
     Description
     -----------
        Script wrapper variables are obfuscated, a complex password is used and encoded password is saved in registry.
        Only the user that saved the password can decode it as it is hashed with their logon identity. 
        
            
     .Inputs
        None
     .Outputs
        none
            
 
             
 #>
   
     [CmdletBinding(SupportsShouldProcess = $false)] 
 
     param(           
                         
         [Parameter( 
         Mandatory=$true,
         Position=0,
         ValueFromPipeline=$false)]
         [string]$inputScript,   
       
         [Parameter( 
         Mandatory=$false,
         Position=1,
         ValueFromPipeline=$false)]
         [string]$password="Password1",      
                   
         [Parameter( 
         Mandatory=$false,
         ValueFromPipeline=$false)]
         [switch]$perUser,
       
         [Parameter( 
         Mandatory=$false,
         ValueFromPipeline=$false)]
         [switch]$forceUnC, 
            
         [Parameter( 
         Mandatory=$false,
         ValueFromPipeline=$false)]
         [switch]$savePass,
             
         [Parameter( 
         Mandatory=$false,
         ValueFromPipeline=$false)]
         [switch]$test,
             
         [Parameter( 
         Mandatory=$false,
         ValueFromPipeline=$false)]
         [switch]$newTemplate,
             
         [Parameter( 
         Mandatory=$false,
         ValueFromPipeline=$false)]
         [switch]$obfuscate,
             
         [Parameter( 
         Mandatory=$false,
         ValueFromPipeline=$false)]
         [string]$outputScript
      )
     
 
     BEGIN 
     {
         $template = "5B5265666C656374696F6E2E417373656D626C795D3A3A4C6F6164576974685061727469616C4E616D65282253797374656D2E536563757269747922297C6F75742D6E756C6C3B202066756E6374696F6E204D4435486173687B20706172616D285B737472696E675D24696E537472696E672C5B7377697463685D2468617368293B2024696E537472696E672B3D285B737472696E675D5B6D6174685D3A3A5049292E737562737472696E6728302C3136293B20246148423D286E65772D4F626A6563742053656375726974792E43727970746F6772617068792E4D443543727970746F5365727669636550726F7669646572292E436F6D70757465486173682824285B436861725B5D5D24696E537472696E6729293B206966282468617368297B666F726561636828246279746520696E2024614842297B2474656D703D227B303A587D22202D662024627974653B6966282474656D702E6C656E6774682D657131297B2474656D703D22302474656D70227D3B24726573756C742B3D2474656D707D3B2D6A6F696E2024726573756C743B7D656C73657B246148427D20207D202066756E6374696F6E204465637279707442797465737B20706172616D282453796D416C672C24696E42797465732C24526567486578293B20247866726D3D2453796D416C672E437265617465446563727970746F7228293B207472797B246F7574426C6F636B3D247866726D2E5472616E73666F726D46696E616C426C6F636B2824696E42797465732C20302C2024696E42797465732E4C656E677468297D2063617463687B77726974652D686F7374202257726F6E67205057223B476574526567202D6B65792024526567486578202D56616C756520246E756C6C202D7365743B627265616B7D2072657475726E205B53797374656D2E546578742E556E69636F6465456E636F64696E675D3A3A556E69636F64652E476574537472696E6728246F7574426C6F636B29207D202066756E6374696F6E204857486173687B20706172616D2824536372506174682C5B7377697463685D2450657255736572293B2024617A3D4D443548617368202453637250617468202D686173683B2024627A3D4D443548617368202867776D6920285B53797374656D2E546578742E456E636F64696E675D3A3A555446382E476574537472696E67285B53797374656D2E436F6E766572745D3A3A46726F6D426173653634537472696E67282256326C754D7A4A6655336C7A644756745257356A6247397A64584A6C222929297C257B245F2E285B53797374656D2E546578742E456E636F64696E675D3A3A555446382E476574537472696E67285B53797374656D2E436F6E766572745D3A3A46726F6D426173653634537472696E6728225532567961574673546E5674596D5679222929297D29202D686173683B2024637A3D4D4435486173682028676320245363725061746829202D686173683B2024647A3D4D443548617368202824686F73742E6E616D6529202D686173683B2024657A3D584F52486173682028584F52486173682024617A2024627A292028584F52486173682024637A2024647A293B206966282450657255736572297B24647A3D4D4435486173682028436F6E7665727446726F6D2D536563757265537472696E67202D536563757265537472696E672028436F6E76657274546F2D536563757265537472696E67202D737472696E672024657A202D6173706C61696E74657874202D666F72636529292D686173683B584F52486173682024657A2024647A7D656C73657B24657A7D207D202066756E6374696F6E2072756E5363726970747B20706172616D285B737472696E675D24746578746F66736372697074626C6F636B293B2024657865637574696F6E636F6E746578742E496E766F6B65436F6D6D616E642E4E6577536372697074426C6F636B2824746578746F66736372697074626C6F636B29207D202066756E6374696F6E20584F52486173687B20706172616D285B737472696E675D24696E707574312C205B737472696E675D24696E70757432293B2024636F6D62696E656442797465733D4028293B20666F722824636F756E743D303B24636F756E742D6C7424696E707574312E6C656E6774683B24636F756E742B2B297B2024686578313D24696E707574312E737562737472696E672824636F756E742C31293B24686578323D24696E707574322E737562737472696E672824636F756E742C31293B2024626974313D5B436F6E766572745D3A3A546F537472696E67282230782468657831222C32293B24626974313D222428223022202A28342D2824626974312E6C656E6774682929292462697431223B2024626974323D5B436F6E766572745D3A3A546F537472696E67282230782468657832222C32293B24626974323D222428223022202A28342D2824626974322E6C656E6774682929292462697432223B2024636F6D62696E6564426974733D4028293B20666F722824636F756E74323D303B24636F756E74322D6C74343B24636F756E74322B2B297B20202478313D24626974312E737562737472696E672824636F756E74322C31293B2478323D24626974322E737562737472696E672824636F756E74322C31293B2024636F6D62696E6564426974732B3D247831202D62786F72202478323B202466696E616C3D5B436F6E766572745D3A3A546F496E74363428282D6A6F696E2024636F6D62696E656442697473292C203229207D20246865783D227B303A587D22202D66282466696E616C293B2024636F6D62696E656442797465732B3D24686578207D202D6A6F696E2024636F6D62696E65644279746573207D202066756E6374696F6E2048657841736369697B20706172616D2824646279746573292024636F756E743D303B2473746F703D246462797465732E6C656E6774683B246465637279707465643D22223B20646F7B246865783D246462797465732E737562737472696E672824636F756E742C32293B20246465637279707465642B3D5B436861725D5B436F6E766572745D3A3A546F496E74333228246865782C3136293B2024636F756E743D24636F756E742B327D7768696C652824636F756E742D6C742473746F70293B20262872756E53637269707420282D6A6F696E20246465637279707465642929207D202046756E6374696F6E204C6963656E63657B20706172616D28245265674865782C247073772C24536372506174682C2450657255736572293B2069662824707377297B476574526567202D6B65792024526567486578202D56616C75652028584F52486173682848574861736820245363725061746820245065725573657229202470737729202D5365747D20656C73657B247265673D476574526567202D6B657920245265674865783B69662824726567297B584F5248617368284857486173682024536372506174682024506572557365722920247265677D7D207D202046756E6374696F6E204765745265677B20706172616D285B737472696E675D246B65792C5B737472696E675D2476616C75652C5B7377697463685D247365742C5B737472696E675D247265673D285B53797374656D2E546578742E456E636F64696E675D3A3A555446382E476574537472696E67285B53797374656D2E436F6E766572745D3A3A46726F6D426173653634537472696E67282253457444565470635532396D64486468636D56635256425422292929293B206966282128746573742D70617468202452656729297B4E49202D706174682024526567202D666F7263657C6F75742D6E756C6C3B53502024526567202D6E20246B6579202D766120246E756C6C7D2069662824736574297B53502024526567202D6E20246B6579202D7661202476616C75657D656C73657B7472797B72657475726E2847502024526567202D6E20244B6579202D4541202773746F7027292E246B65797D63617463687B7D7D207D20205B737472696E675D24536372506174683D222428244D79496E766F636174696F6E2E4D79436F6D6D616E642E446566696E6974696F6E29223B2069662824666F726365556E63297B696628212824536372506174682E737562737472696E6728302C32292D6571225C5C2229297B224E6F742052756E2046726F6D20554E43223B627265616B7D7D20245265674865783D584F524861736820284D443548617368202824656E634865782E737562737472696E6728312C33322929202D68617368292028485748617368202453637250617468202450657255736572293B2020696628247361766550617373297B24706173733D4C6963656E6365202452656748657820246E756C6C2024536372506174682024506572557365723B6966282128247061737329297B24706173733D4D44354861736828526561642D486F73742022456E7465722050617373776F72642229202D686173683B6C6963656E636520245265674865782024706173732024536372506174682024506572557365727D7D20656C73657B24706173733D4D44354861736828526561642D486F73742022456E7465722050617373776F72642229202D686173687D2020244165734353503D4E65772D4F626A6563742053797374656D2E53656375726974792E43727970746F6772617068792E41657343727970746F5365727669636550726F76696465723B20244165734353502E6B65793D4D4435486173682024706173733B20244165734353502E69763D4D443548617368284D443548617368202470617373202D68617368293B202024636F756E743D303B2473746F703D24456E634865782E6C656E6774683B24456E6342797465733D4028293B20646F7B246865783D24456E634865782E737562737472696E672824636F756E742C32293B20247A7A3D5B436F6E766572745D3A3A546F496E74333228246865782C3136293B2024456E6342797465732B3D5B53797374656D2E436F6E766572745D3A3A546F4279746528247A7A293B2024636F756E743D24636F756E742B327D7768696C652824636F756E742D6C742473746F70293B20204865784173636969202844656372797074427974657320244165734353502024456E6342797465732024526567486578292020202020202020";
         
         [Reflection.Assembly]::LoadWithPartialName("System.Security") | out-null;
 
         function new-scriptblock
         {
             param([string]$textofscriptblock);
             $executioncontext.InvokeCommand.NewScriptBlock($textofscriptblock)
         }
 
         function MD5-Hash
         {
             param ([string]$inString,[switch]$hash);   
             $inString+=([string][math]::PI).substring(0,16);
             $aHB=(new-Object Security.Cryptography.MD5CryptoServiceProvider).ComputeHash($([Char[]]$inString));  
             if ($hash){foreach ($byte in $aHB){$temp="{0:X}" -f $byte;if ($temp.length-eq1){$temp="0$temp"};$result+=$temp};-join $result;}else{$aHB};         
         }
 
         function Encrypt-String
         {
            param($SymAlg, [string]$inString)           
            $inBlock = [System.Text.UnicodeEncoding]::Unicode.getbytes($instring)
            $xfrm = $symAlg.CreateEncryptor()
            $outBlock = $xfrm.TransformFinalBlock($inBlock, 0, $inBlock.Length);
            return $outBlock;
         }
 
         function Decrypt-Bytes 
         {
            param($SymAlg, $inBytes)           
            $xfrm = $symAlg.CreateDecryptor();
            $outBlock = $xfrm.TransformFinalBlock($inBytes, 0, $inBytes.Length)
            return [System.Text.UnicodeEncoding]::Unicode.GetString($outBlock)
         }        
         
         function HexToAscii
         {
             param($dHex);
             $count=0;$stop=$dHex.length;$dAscii=@();
             do{
                 $hex=$dHex.substring($count,2);
                 $dAscii+=[Char][Convert]::ToInt32($hex,16);
                 $count=$count+2;      
             }while($count-lt$stop);
             [string]$result = -join $dAscii
             $result
         }
     }
     
     PROCESS
     {
 
 
         $pass = MD5-Hash $password -hash
 
 
         if (Test-Path $inputScript){[string]$inputScriptContent = get-content $inputScript}
         else {"Can't read $inputScript"; break}
 
 
         $hex2=$inputScriptContent.ToCharArray() | % {"{0:X}" -f ([int][char]$_)} | % {if ($_.length -eq 1){"0"+$_}else{$_}} 
         $strHex = -join $hex2
 
 
         if ($newTemplate){$strhex; break}
 
 
         $AesCSP = New-Object System.Security.Cryptography.AesCryptoServiceProvider
         $AesCSP.key = MD5-Hash $pass
         $AesCSP.iv = MD5-Hash (MD5-Hash $pass -hash)
 
 
         $EncBytes = Encrypt-String $aesCSP $strHex
 
 
         $EncHex = [System.Convert]::ToBase64String($EncBytes)
         $EncHex = -join ($EncBytes | % {$hh="{0:X}" -f $_; if ($hh.length-eq1){"0$hh"}else{$hh}})
 
 
         $count=0;$stop=$EncHex.length;$EncBytes2=@()
         do{
             $hex=$EncHex.substring($count,2);
             $zz=[Convert]::ToInt32($hex,16);
             $EncBytes2+=[System.Convert]::ToByte($zz);
             $count=$count+2;    
         }while($count-lt$stop);
 
 
         $dHex = Decrypt-Bytes $AesCSP $EncBytes2;
 
 
         $sb=HexToAscii $dHex 
 
 
         if ($test)
         {
             &(new-scriptblock $sb)
         }
         else
         {
         
         
         if (!($outputScript)){$outputScript="$($EncHex.substring(0,15))-$password.ps1"}        
 
         $tb="`$encHex=`"$encHex`";$(HexToAscii $template)"
 
         if($peruser){$tb="`$peruser=`$true;"+$tb}else{$tb="`$peruser=`$false;"+$tb}
         if($forceUnC){$tb="`$forceUnC=`$true;"+$tb}else{$tb="`$forceUnC=`$false;"+$tb}
         if($savePass){$tb="`$savePass=`$true;"+$tb}else{$tb="`$savePass=`$false;"+$tb}
 
         if ($obfuscate)
         {
             $sub = 'FFF,MD5Hash,$FFE,$inString,$FFD,$hash,-FFD,-hash,$FFC,$aHB,$FFB,$byte,$FFA,$temp,$FF9,$result,FF8,DecryptBytes,$FF7,$SymAlg,$FF6,$inBytes,$FF5,$RegHex,$FF4,$xfrm,$xxx,$xxx,`
             $xxx,$xxx,$FF3,$outBlock,FF2,HWHash,$FF1,$ScrPath,$FF0,$PerUser,$FF0,$peruser,$FEF,$az,$FEE,$bz,$FED,$cz,$FEC,$dz,$FEB,$ez,FEA,runScript,$FE9,$textofscriptblock,FE8,XORHash,$xxx,$xxx,`
             $xxx,$xxx,$FE7,$input1,$FE6,$input2,$FE5,$combinedBytes,$FE4,$count2,$FE3,$hex1,$FE2,$hex2,$FE1,$bit1,$FE0,$bit2,$FDF,$combinedBits,$FDE,$count,$FDD,$x1,$FDC,$x2,$xxx,$xxx,`
             $xxx,$xxx,$FDB,$final,FD4,HexAscii,$FD3,$dbytes,$FD2,$stop,$FD1,$decrypted,FD0,Licence,FD0,licence,$FCF,$RegHex,$FCE,$psw,$FCD,$ScrPath,FCB,GetReg,$xxx,$xxx,`
             $xxx,$xxx,FC7,GetReg,$FC6,$forceUnc,$FC6,$forceUnC,$FC5,$encHex,$FC5,$EncHex,$FC4,$savePass,$FC3,$pass,$FC2,$AesCSP,$FC1,$hex,$FC0,$EncBytes,$xxx,$xxx,`
             $xxx,$xxx,FBF,value,$FBF,$value,FBF,Value,$FBD,$Reg,$FBD,$reg,$FBC,$key,$FBC,$Key,-FBC,-key,$FBB,$Set,$FBB,$set,-FBB,-Set,-FBB,-set,$FBA,$zz,$xxx,$xxx,`
             $FB9,$xxx,`
             $FB8,$xxx,`
             $FB7,$xxx,`
             $FB6,$xxx,`
             $FB5,$xxx,`
             $FB4,$xxx,`
             $FB3,$xxx,`
             $FB2,$xxx,`
             $FB1,$xxx,`
             $FB0,$xxx,`
             $FAF,$xxx,`
             $FAE,$xxx,`
             $FAD,$xxx,`
             $FAC,$xxx,`
             $FAB,$xxx,`
             $FAA,$xxx,`
             $FA9,$xxx,`
             $FA8,$xxx,`
             $FA7,$xxx,`
             $FA6,$xxx,`
             $FA5,$xxx,`
             $FA4,$xxx,`
             $FA3,$xxx,`
             $FA2,$xxx,`
             $FA1,$xxx,`
             $FA0,$xxx,`
             $F9F,$xxx,`
             $F9E,$xxx,`
             $F9D,$xxx,`
             $F9C,$xxx,`
             $F9B,$xxx,`
 
             $subArray = $sub.Split(',') 
             for($x=0;$X-le$subArray.length-2; $x=$x+2){$tb=$tb.replace($subArray[$x+1],$subArray[$x])}
         }
 
         $tb | out-file $outputScript
         ii $outputScript
 
         }
     }
 }
 
 
 
 
 
 <#
 
 Challenge1
 Create a new encrypted script, just use -savePass option.
 Run script (using console), thus saving password in registry. 
 Then without using original password or modification to script, obtain plain text of encrypted script.  
 Use debuger tracer or any other idea or external tool to achieve this goal.
 Full points for explaining how you succeeded. Skip to Challenge 4
 
 Challenge2 
 Create a new encrypted script, just use -savePass option.
 Run script (using console), thus saving password in registry. 
 Move or rename script and then
 Manually build the hardware hash by analysing code and original environment 
 extract password hash from registry key using your hacked hardware hash
 and then use that to obtain plain text of encrypted script. 
 Re-entering password "Password1" at any time is a fail. (A true attacker would not know it)
 
 Challenge3
 Same as Challenge2 but also use -perUser option 
 Instead of moving script, log onto computer using another account and attempt to retrieve plain text of decrypted script.
 Re-entering password "Password1" at any time is a fail. (A true attacker would not know it)
 Simulating original user's password being reset by logging on as them to get the hash of their identity is a half pass. 
 Explain how you obtained the user specific part of the hash without compromising users account for full points
 
 Challenge4 
 Move to the next level by modifying the script to make it more secure.
 Explain the steps or provide code for full points.
 
 Challenge5
 You are obviously interested in the concept of this script to have got this far, 
 add another feature, correct a bug, make it more robust and/or repost to another forum to see if others can take the idea further. 
 Thanks.   
 
 #>
`

