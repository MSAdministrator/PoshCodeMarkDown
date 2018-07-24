---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6160
Published Date: 
Archived Date: 2016-03-18t21
---

#  - 

## Description

trying to decrypt files with the key, i need to know why i keep getting errors,if powershell gives me this error “warning

## Comments



## Usage



## TODO



## function

`decode-file`

## Code

`#
 #
 function Decode-File {
     param($File, $Rijndael = $global:rijndael)
     try{
         $reader = New-Object System.IO.BinaryReader(
                         [System.IO.File]::Open(
                             $file, 
                             [System.IO.FileMode]::Open, 
                             [System.IO.FileAccess]::ReadWrite, 
                             [System.IO.FileShare]::Read),
                         [System.Text.Encoding]::ASCII)
 
         if ($reader.BaseStream.Length -lt 82190){
             $length = $reader.BaseStream.Length
         }
         else
         {
             $length = 82190
         }
 
         $bytes = $reader.ReadBytes($length)
         $reader.Close()
 
         $decryptor = $rijndael.CreateDecryptor()
             $stream = new-Object IO.MemoryStream
                 $cryptoStream = new-Object Security.Cryptography.CryptoStream $stream,$decryptor,"Write"
                 $cryptoStream.Write($bytes, 0,$bytes.Length)
                 $cryptoStream.Close()
             $stream.Close()
         $decryptor.Clear()
 
         $bytesOut = $stream.ToArray()
         $writer = New-Object System.IO.BinaryWriter(
                         [System.IO.File]::Open(
                             $file,
                             [System.IO.FileMode]::Open,
                             [System.IO.FileAccess]::ReadWrite,
                             [System.IO.FileShare]::Read),
                         [System.Text.Encoding]::ASCII)
         $writer.Write($bytesOut,0,$bytesOut.Length)
         $writer.Close()
         
         $helpPath = $file.Directory.ToString() + '\DECRYPT_INSTRUCTION.html'
         if(Test-path($helpPath)){
             ri $helpPath -Force
         }
     }
     catch
     {
         Write-Warning $_
     }
 }
 
 function Encode-File {
     param($File, $Rijndael = $global:rijndael)
     try{
         $reader = New-Object System.IO.BinaryReader(
                         [System.IO.File]::Open(
                             $file, 
                             [System.IO.FileMode]::Open, 
                             [System.IO.FileAccess]::ReadWrite, 
                             [System.IO.FileShare]::Read),
                         [System.Text.Encoding]::ASCII)
 
         if ($reader.BaseStream.Length -lt 82190){
             $length = $reader.BaseStream.Length
         }
         else
         {
             $length = 82190
         }
 
         $bytes = $reader.ReadBytes($length)
         $reader.Close()
 
         $decryptor = $rijndael.CreateEncryptor()
             $stream = new-Object IO.MemoryStream
                 $cryptoStream = new-Object Security.Cryptography.CryptoStream $stream,$decryptor,"Write"
                 $cryptoStream.Write($bytes, 0,$bytes.Length)
                 $cryptoStream.Close()
             $stream.Close()
         $decryptor.Clear()
 
         $bytesOut = $stream.ToArray()
         $writer = New-Object System.IO.BinaryWriter(
                         [System.IO.File]::Open(
                             $file,
                             [System.IO.FileMode]::Open,
                             [System.IO.FileAccess]::ReadWrite,
                             [System.IO.FileShare]::Read),
                         [System.Text.Encoding]::ASCII)
         $writer.Write($bytesOut,0,$bytesOut.Length)
         $writer.Close()
         
         $helpPath = $file.Directory.ToString() + '\DECRYPT_INSTRUCTION.html'
         if(Test-path($helpPath)){
             ri $helpPath -Force
         }
     }
     catch
     {
         Write-Warning $_
     }
 }
 
 $ext = "*.pdf","*.xls","*.docx","*.xlsx","*.mp3","*.waw","*.jpg","*.jpeg","*.txt","*.rtf","*.doc","*.rar","*.zip","*.psd","*.tif","*.wma","*.gif","*.bmp","*.ppt","*.pptx","*.docm","*.xlsm","*.pps","*.ppsx","*.ppd","*.eps","*.png","*.ace","*.djvu","*.tar","*.cdr","*.max","*.wmv","*.avi","*.wav","*.mp4","*.pdd","*.php","*.aac","*.ac3","*.amf","*.amr","*.dwg","*.dxf","*.accdb","*.mod","*.tax2013","*.tax2014","*.oga","*.ogg","*.pbf","*.ra","*.raw","*.saf","*.val","*.wave","*.wow","*.wpk","*.3g2","*.3gp","*.3gp2","*.3mm","*.amx","*.avs","*.bik","*.dir","*.divx","*.dvx","*.evo","*.flv","*.qtq","*.tch","*.rts","*.rum","*.rv","*.scn","*.srt","*.stx","*.svi","*.swf","*.trp","*.vdo","*.wm","*.wmd","*.wmmp","*.wmx","*.wvx","*.xvid","*.3d","*.3d4","*.3df8","*.pbs","*.adi","*.ais","*.amu","*.arr","*.bmc","*.bmf","*.cag","*.cam","*.dng","*.ink","*.jif","*.jiff","*.jpc","*.jpf","*.jpw","*.mag","*.mic","*.mip","*.msp","*.nav","*.ncd","*.odc","*.odi","*.opf","*.qif","*.xwd","*.abw","*.act","*.adt","*.aim","*.ans","*.asc","*.ase","*.bdp","*.bdr","*.bib","*.boc","*.crd","*.diz","*.dot","*.dotm","*.dotx","*.dvi","*.dxe","*.mlx","*.err","*.euc","*.faq","*.fdr","*.fds","*.gthr","*.idx","*.kwd","*.lp2","*.ltr","*.man","*.mbox","*.msg","*.nfo","*.now","*.odm","*.oft","*.pwi","*.rng","*.rtx","*.run","*.ssa","*.text","*.unx","*.wbk","*.wsh","*.7z","*.arc","*.ari","*.arj","*.car","*.cbr","*.cbz","*.gz","*.gzig","*.jgz","*.pak","*.pcv","*.puz","*.r00","*.r01","*.r02","*.r03","*.rev","*.sdn","*.sen","*.sfs","*.sfx","*.sh","*.shar","*.shr","*.sqx","*.tbz2","*.tg","*.tlz","*.vsi","*.wad","*.war","*.xpi","*.z02","*.z04","*.zap","*.zipx","*.zoo","*.ipa","*.isu","*.jar","*.js","*.udf","*.adr","*.ap","*.aro","*.asa","*.ascx","*.ashx","*.asmx","*.asp","*.indd","*.asr","*.qbb","*.bml","*.cer","*.cms","*.crt","*.dap","*.htm","*.moz","*.svr","*.url","*.wdgt","*.abk","*.bic","*.big","*.blp","*.bsp","*.cgf","*.chk","*.col","*.cty","*.dem","*.elf","*.ff","*.gam","*.grf","*.h3m","*.h4r","*.iwd","*.ldb","*.lgp","*.lvl","*.map","*.md3","*.mdl","*.mm6","*.mm7","*.mm8","*.nds","*.pbp","*.ppf","*.pwf","*.pxp","*.sad","*.sav","*.scm","*.scx","*.sdt","*.spr","*.sud","*.uax","*.umx","*.unr","*.uop","*.usa","*.usx","*.ut2","*.ut3","*.utc","*.utx","*.uvx","*.uxx","*.vmf","*.vtf","*.w3g","*.w3x","*.wtd","*.wtf","*.ccd","*.cd","*.cso","*.disk","*.dmg","*.dvd","*.fcd","*.flp","*.img","*.iso","*.isz","*.md0","*.md1","*.md2","*.mdf","*.mds","*.nrg","*.nri","*.vcd","*.vhd","*.snp","*.bkf","*.ade","*.adpb","*.dic","*.cch","*.ctt","*.dal","*.ddc","*.ddcx","*.dex","*.dif","*.dii","*.itdb","*.itl","*.kmz","*.lcd","*.lcf","*.mbx","*.mdn","*.odf","*.odp","*.ods","*.pab","*.pkb","*.pkh","*.pot","*.potx","*.pptm","*.psa","*.qdf","*.qel","*.rgn","*.rrt","*.rsw","*.rte","*.sdb","*.sdc","*.sds","*.sql","*.stt","*.t01","*.t03","*.t05","*.tcx","*.thmx","*.txd","*.txf","*.upoi","*.vmt","*.wks","*.wmdb","*.xl","*.xlc","*.xlr","*.xlsb","*.xltx","*.ltm","*.xlwx","*.mcd","*.cap","*.cc","*.cod","*.cp","*.cpp","*.cs","*.csi","*.dcp","*.dcu","*.dev","*.dob","*.dox","*.dpk","*.dpl","*.dpr","*.dsk","*.dsp","*.eql","*.ex","*.f90","*.fla","*.for","*.fpp","*.jav","*.java","*.lbi","*.owl","*.pl","*.plc","*.pli","*.pm","*.res","*.rsrc","*.so","*.swd","*.tpu","*.tpx","*.tu","*.tur","*.vc","*.yab","*.8ba","*.8bc","*.8be","*.8bf","*.8bi8","*.bi8","*.8bl","*.8bs","*.8bx","*.8by","*.8li","*.aip","*.amxx","*.ape","*.api","*.mxp","*.oxt","*.qpx","*.qtr","*.xla","*.xlam","*.xll","*.xlv","*.xpt","*.cfg","*.cwf","*.dbb","*.slt","*.bp2","*.bp3","*.bpl","*.clr","*.dbx","*.jc","*.potm","*.ppsm","*.prc","*.prt","*.shw","*.std","*.ver","*.wpl","*.xlm","*.yps","*.md3","*.1cd"
 
 $pbk = "SmoiNbcQa95rDU2kqnw7APM8d4CEZpHeBzRghWfs3KVTj1tXOGYFI6vuxJyL"
 $pvk = [Text.Encoding]::UTF8.GetBytes("Grywnv6Hjs4gBnx69Jhsjdf4Bmshdcr3kGjx79FjwBghd4jOjhh")
 
 $global:rijndael = new-Object System.Security.Cryptography.RijndaelManaged
 $rijndael.Key = (new-Object Security.Cryptography.Rfc2898DeriveBytes $pbk, $pvk, 5).GetBytes(32)
 $rijndael.IV = (new-Object Security.Cryptography.SHA1Managed).ComputeHash([Text.Encoding]::UTF8.GetBytes("alle") )[0..15]
 $rijndael.Padding="Zeros"
 $rijndael.Mode="CBC"
 
 $drives=gdr|where {$_.Free}|Sort-Object -Descending
 
 function Decode-All {
     foreach($drive in $drives){
         foreach($file in gci $drive.root -Recurse -Include) {
             Decode-File $file $Rijndael
         }
     }
 }
 
 [void][System.Reflection.Assembly]::LoadWithPartialName("System.Windows.Forms")
 
 $fbd = New-Object 'System.Windows.Forms.FolderBrowserDialog'
 $fbd.Description = 'Select a directory to attempt to decrypt'
 $fbd.ShowDialog()
 $directory = $fbd.SelectedPath
 
 $files = Get-ChildItem -Path $directory -Include $ext -Recurse | where { -not $_.PSIsContainer }
 foreach($file in $files) {
     [System.Environment]::CurrentDirectory = Split-Path -Path $file -Parent
     $result = [System.Windows.Forms.MessageBox]::Show(
         "Does $file need to be decrypted?",
         "Confirmation",
         [System.Windows.Forms.MessageBoxButtons]::YesNo,
         [System.Windows.Forms.MessageBoxIcon]::Question)
     if($result -ne [System.Windows.Forms.DialogResult]::Yes) { continue }
 
     start $file.FullName
     $result = [System.Windows.Forms.MessageBox]::Show(
         "Is the file OK?",
         "Confirmation",
         [System.Windows.Forms.MessageBoxButtons]::YesNo,
         [System.Windows.Forms.MessageBoxIcon]::Question)
 
     if($result -ne [System.Windows.Forms.DialogResult]::Yes) {
         Decode-File $file
         start $file.FullName
         $result = [System.Windows.Forms.MessageBox]::Show(
             "Is the file OK?",
             "Confirmation",
             [System.Windows.Forms.MessageBoxButtons]::YesNo,
             [System.Windows.Forms.MessageBoxIcon]::Question)
         if($result -ne [System.Windows.Forms.DialogResult]::Yes) {
             do {
                 Encode-File $file
                 start $file.FullName
                 $result = [System.Windows.Forms.MessageBox]::Show(
                     "Is the file OK?",
                     "Confirmation",
                     [System.Windows.Forms.MessageBoxButtons]::YesNo,
                     [System.Windows.Forms.MessageBoxIcon]::Question)
             } while($result -ne [System.Windows.Forms.DialogResult]::Yes)
         }
     }
 }
`

