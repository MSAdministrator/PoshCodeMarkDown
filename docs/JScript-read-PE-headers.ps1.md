---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5030
Published Date: 2014-03-30t18
Archived Date: 2014-04-03t07
---

# jscript - 

## Description

see original post at https

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 var pe = pe || {
   getRawData : function(fileName) {
     try {
       var xml = new ActiveXObject('Microsoft.XMLDOM').createElement('Binary'),
           raw, e_magic, e_lfanew;
       xml.dataType = 'bin.hex';
       
       with (new ActiveXObject('ADODB.Stream')) {
         Type = 1;
         Open();
         
         LoadFromFile(fileName);
         xml.nodeTypedValue = Read(1024);
         Close();
       }
       
       raw = xml.text.match(/.{2}/g);
       
       e_magic  = parseInt('0x' + raw.slice(0, 2).reverse().join(''));
       e_lfanew = 256 * parseInt('0x' + raw[0x3D]) + parseInt('0x' + raw[0x3C]);
       
       if (e_magic != 23117 || raw.slice(e_lfanew, (e_lfanew + 4)).join('') != 50450000) {
         WScript.echo('Invalid file format.');
         WScript.Quit(1);
       }
     }
     catch (e) {
       WScript.echo(e.message);
       WScript.Quit(1);
     }
     
     return [raw, e_lfanew];
   },
   
   getDosHeader : function(fileName) {
     Array.prototype.getBytes = function(b) {
       return parseInt('0x' + this.slice(b, b + 2).reverse().join(''));
     }
     
     var raw = this.getRawData(fileName)[0],
         IMAGE_DOS_HEADER = {
           'e_magic'    : raw.getBytes(0x00),
           'e_cblp'     : raw.getBytes(0x02),
           'e_cp'       : raw.getBytes(0x04),
           'e_crlc'     : raw.getBytes(0x06),
           'e_cparhdr'  : raw.getBytes(0x08),
           'e_minalloc' : raw.getBytes(0x0A),
           'e_maxalloc' : 256 * parseInt('0x' + raw[0x0D]) + parseInt('0x' + raw[0x0C]),
           'e_ss'       : raw.getBytes(0x0E),
           'e_sp'       : raw.getBytes(0x10),
           'e_csum'     : raw.getBytes(0x12),
           'e_ip'       : raw.getBytes(0x14),
           'e_cs'       : raw.getBytes(0x16),
           'e_lfarlc'   : raw.getBytes(0x18),
           'e_ovno'     : raw.getBytes(0x1A),
           'e_res'      : raw.slice(0x1C, 0x23),
           'e_oemid'    : raw.getBytes(0x24),
           'e_oeminfo'  : raw.getBytes(0x26),
           'e_res2'     : raw.slice(0x28, 0x3B),
           'e_lfanew'   : this.getRawData(fileName)[1]
         }
     
     this.print(IMAGE_DOS_HEADER, 11);
   },
   
   getFileHeader : function(fileName) {
     var raw = this.getRawData(fileName),
         i, mac = {
           'I386'  : 0x014c,
           'IA64'  : 0x0200,
           'AMD64' : 0x8664
         },
         ifc = {
           'RelocationsStripped'      : 0x0001,
           'Executable'               : 0x0002,
           'LineNumbersStripped'      : 0x0004,
           'SymbolsStripped'          : 0x0008,
           'AggressiveTrimWorkingSet' : 0x0010,
           'LargeAddressAware'        : 0x0020,
           'Supports16Bit'            : 0x0040,
           'ReservedBytesWo'          : 0x0080,
           'Supports32Bit'            : 0x0100,
           'DebugInfoStripped'        : 0x0200,
           'RunFromSwapIfInRmvbl'     : 0x0400,
           'RunFromSwapIfInNetwrk'    : 0x0800,
           'SystemFile'               : 0x1000,
           'Dll'                      : 0x2000,
           'OnlyFroSingleCoreProc'    : 0x4000,
           'BytesOfWordReserved'      : 0x8000
         },
         IMAGE_FILE_HEADER = {};
     
     Array.prototype.getBytes = function(b, e) {
       return parseInt('0x' + this.slice(raw[1] + b, raw[1] + b + e).reverse().join(''));
     }
     
     for (i in mac)
       if (raw[0].getBytes(0x04, 2) == mac[i]) IMAGE_FILE_HEADER['Machine'] = i;
     
     with (new Date(1970, 0, 1)) {
       IMAGE_FILE_HEADER['TimeDateStamp'] = new Date(
         setSeconds(getSeconds() + raw[0].getBytes(0x08, 4))
       );
     }
     
     IMAGE_FILE_HEADER['PointerToSymbolTable'] = raw[0].getBytes(0x0C, 2);
     IMAGE_FILE_HEADER['NumberOfSymbols']      = raw[0].getBytes(0x10, 2);
     IMAGE_FILE_HEADER['SizeOfOptionalHeader'] = raw[0].getBytes(0x14, 2).toString(16);
     IMAGE_FILE_HEADER['Characteristics']      = '';
     for (i in ifc) {
       if ((raw[0].getBytes(0x16, 4) & ifc[i]) == ifc[i])
         IMAGE_FILE_HEADER['Characteristics'] += i + '; ';
     }
     
     this.print(IMAGE_FILE_HEADER, 21);
   },
   
   getOptionalHeader : function(fileName) {
     var raw = this.getRawData(fileName),
         i, iss = {
           'Unknown'           : 0x00,
           'Native'            : 0x01,
           'WindowsGUI'        : 0x02,
           'WindowsCUI'        : 0x03,
           'OS2_CUI'           : 0x04,
           'POSIX_CUI'         : 0x05,
           'NativeWindows'     : 0x06,
           'WindowsCE_CUI'     : 0x07,
           'EFIApplication'    : 0x08,
           'EFIBootServiceDrv' : 0x09,
           'EFIRuntimeDrv'     : 0x0A,
           'EFIRom'            : 0x0B,
           'XBox'              : 0x0C,
           'WinBootApp'        : 0x10
         },
         idc = {
           'Res0'             : 0x0001,
           'Res1'             : 0x0002,
           'Res2'             : 0x0004,
           'Res3'             : 0x0008,
           'DynamicBase'      : 0x0040,
           'ForceItegrity'    : 0x0080,
           'NxCompatible'     : 0x0100,
           'NoIsolation'      : 0x0200,
           'NoSEH'            : 0x0400,
           'NoBind'           : 0x0800,
           'Res4'             : 0x1000,
           'WDMDriver'        : 0x2000,
           'TerminalSrvAware' : 0x8000
         };
     
     Array.prototype.getBytes = function(b, e) {
       return '0x' + this.slice(raw[1] + b, raw[1] + b + e).reverse().join('');
     }
     
     var bit = (parseInt(raw[0].getBytes(0x16, 4)) & 0x0100) == 0x0100 ? 0 : 1;
     
     var IMAGE_OPTIONAL_HEADER = {
       'Magic'               : raw[0].getBytes(0x18, 2) == 0x10B ? 'PE32' : 'PE32+',
       'Linker'              : [parseInt(raw[0].getBytes(0x1A, 1)), parseInt(raw[0].getBytes(0x1B, 2))],
       'SizeOfCode'          : raw[0].getBytes(0x1C, 4),
       'SizeOfInitData'      : raw[0].getBytes(0x20, 4),
       'SizeOfUnInitData'    : raw[0].getBytes(0x24, 4),
       'AddressOfEntryPoint' : raw[0].getBytes(0x28, 4),
       'BaseOfCode'          : raw[0].getBytes(0x2C, 4),
       'BaseOfData'          : bit == 0 ? raw[0].getBytes(0x30, 4) : 'n/a',
       'ImageBase'           : bit == 0 ? raw[0].getBytes(0x34, 4) : raw[0].getBytes(0x30, 8),
       'SectionAlignment'    : raw[0].getBytes(0x38, 4),
       'FileAlignment'       : raw[0].getBytes(0x3C, 4),
       'OSVersion'           : [parseInt(raw[0].getBytes(0x40, 2)), parseInt(raw[0].getBytes(0x42, 2))],
       'ImageVersion'        : [parseInt(raw[0].getBytes(0x44, 2)), parseInt(raw[0].getBytes(0x46, 2))],
       'SubsystemVersion'    : [parseInt(raw[0].getBytes(0x48, 2)), parseInt(raw[0].getBytes(0x4A, 2))],
       'Win32VersionValue'   : raw[0].getBytes(0x4C, 4),
       'SizeOfImage'         : raw[0].getBytes(0x50, 4),
       'SizeOfHeaders'       : raw[0].getBytes(0x54, 4),
       'Checksum'            : raw[0].getBytes(0x58, 4)
     };
     
     for (i in iss) {
       if (raw[0].getBytes(0x5C, 2) == iss[i])
         IMAGE_OPTIONAL_HEADER['Subsystem'] = i;
     }
     IMAGE_OPTIONAL_HEADER['DllCharacteristics'] = '';
     for (i in idc) {
       if ((raw[0].getBytes(0x5E, 4) & idc[i]) == idc[i])
         IMAGE_OPTIONAL_HEADER['DllCharacteristics'] += i + '; ';
     }
     
     IMAGE_OPTIONAL_HEADER['SizeOfStackreserve']  =
       bit == 0 ? raw[0].getBytes(0x60, 4) : raw[0].getBytes(0x60, 8);
     IMAGE_OPTIONAL_HEADER['SizeOfStackCommit']   =
       bit == 0 ? raw[0].getBytes(0x64, 4) : raw[0].getBytes(0x68, 8);
     IMAGE_OPTIONAL_HEADER['SizeOfHeapReserve']   =
       bit == 0 ? raw[0].getBytes(0x68, 4) : raw[0].getBytes(0x70, 8);
     IMAGE_OPTIONAL_HEADER['SizeOfHeapCommit']    =
       bit == 0 ? raw[0].getBytes(0x6C, 4) : raw[0].getBytes(0x78, 8);
     IMAGE_OPTIONAL_HEADER['LoaderFlags']         =
       bit == 0 ? raw[0].getBytes(0x70, 4) : raw[0].getBytes(0x80, 4);
     IMAGE_OPTIONAL_HEADER['NumberOfRvaAndSizes'] =
       bit == 0 ? raw[0].getBytes(0x74, 4) : raw[0].getBytes(0x84, 4);
     
     this.print(IMAGE_OPTIONAL_HEADER, 20);
   },
   
   getSectionHeaders : function(fileName) {
     var raw = this.getRawData(fileName),
         ptr = raw[1] + parseInt('0x' + raw[0].slice(raw[1] + 0x14, raw[1] + 0x16)) + 0x18,
         sec, i, isc = {
           'TypeDsect'             : 0x00000001,
           'TypeNoLoad'            : 0x00000002,
           'TypeGroup'             : 0x00000004,
           'TypeNoPadded'          : 0x00000008,
           'TypeCopy'              : 0x00000010,
           'Code'                  : 0x00000020,
           'InitializedData'       : 0x00000040,
           'UninitializedData'     : 0x00000080,
           'LinkOther'             : 0x00000100,
           'LinkInfo'              : 0x00000200,
           'TypeOver'              : 0x00000400,
           'LinkRemove'            : 0x00000800,
           'LinkComDat'            : 0x00001000,
           'Reserved'              : 0x00002000,
           'NoDeferSpecExceptions' : 0x00004000,
           'RelativeGP'            : 0x00008000,
           'MemPurgeable'          : 0x00020000,
           'Memory16Bit'           : 0x00020000,
           'MemoryLocked'          : 0x00040000,
           'MemoryPreload'         : 0x00080000,
           'Align1Bytes'           : 0x00100000,
           'Align2Bytes'           : 0x00200000,
           'Align4Bytes'           : 0x00300000,
           'Align8Bytes'           : 0x00400000,
           'Align16Bytes'          : 0x00500000,
           'Align32Bytes'          : 0x00600000,
           'Align64Bytes'          : 0x00700000,
           'Align128Bytes'         : 0x00800000,
           'Align256Bytes'         : 0x00900000,
           'Align512Bytes'         : 0x00A00000,
           'Align1024Bytes'        : 0x00B00000,
           'Align2048Bytes'        : 0x00C00000,
           'Align4096Bytes'        : 0x00D00000,
           'Align8192Bytes'        : 0x00E00000,
           'AlignMask'             : 0x00F00000,
           'LinkExRelocOverflow'   : 0x01000000,
           'MemoryDiscardable'     : 0x02000000,
           'MemoryNotCached'       : 0x04000000,
           'MemoryNotPaged'        : 0x08000000,
           'MemoryShared'          : 0x10000000,
           'MemoryExecute'         : 0x20000000,
           'MemoryRead'            : 0x40000000,
           'MemoryWrite'           : 0x80000000
         };
     
     Array.prototype.getDetail = function(b) {
       return '0x' + this.slice(ptr + b, ptr + b + 4).reverse().join('');
     }
     
     WScript.echo('Name   VirtSize    VirtAddr    SzRawData   PtrRawData  Characteristics');
     WScript.echo('-----  ----------  ----------  ----------  ----------  ----------------');
     
     for (i = 0; i < parseInt('0x' + raw[0].slice(raw[1] + 0x6, raw[1] + 0x8)); ++i) {
       sec = raw[0].slice(ptr, ptr + 0x07);
       for (var j = 0, cap = '', chr =''; j < sec.length; ++j) {
         cap += String.fromCharCode(parseInt('0x' + sec[j]));
       }
       WScript.StdOut.Write(cap);
       WScript.StdOut.Write(raw[0].getDetail(0x08) + '  ');
       WScript.StdOut.Write(raw[0].getDetail(0x0C) + '  ');
       WScript.StdOut.Write(raw[0].getDetail(0x10) + '  ');
       WScript.StdOut.Write(raw[0].getDetail(0x14) + '  ');
       for (var k in isc) {
         if (parseInt('0x' + raw[0].slice(
           ptr + 0x24, ptr + 0x28).reverse().join('')) & isc[k]) {
           chr += k + '; ';
         }
       }
       WScript.echo(chr);
       ptr += 0x28;
     }
   },
   
   print : function(o, v) {
     var i, j;
     for (i in o) {
       j = i;
       while (j.length != v) j += ' ';
       WScript.echo(j + ': ' + o[i]);
     }
   }
 }
`

