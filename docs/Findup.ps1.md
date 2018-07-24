---
Author: james gentile
Publisher: 
Copyright: 
Email: 
Version: 2.0
Encoding: ascii
License: cc0
PoshCode ID: 3377
Published Date: 2013-04-20t03
Archived Date: 2016-03-05t08
---

# findup - 

## Description

findup – find duplicates c# version. compares files sizes and sha512 hashes to identify duplicates. new regex include/exclude feature. should be compiled with visual studio 11 (beta as of now), as older visual studio c# compilers seem to have a bug that causes crashes on long file names.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 using System;
 using System.Collections;
 using System.Collections.Generic;
 using System.Text;
 using System.Security.Cryptography;
 using System.Runtime.InteropServices;
 using Microsoft.Win32;
 using System.IO;
 using System.Text.RegularExpressions;
 
 
 
 namespace Findup
 {
     public class FileLengthComparer : IComparer<FileInfo>
     {
         public int Compare(FileInfo x, FileInfo y)
         {
             return (x.Length.CompareTo(y.Length));
         }
     }
     
     class Findup
     {
         public static Dictionary<string, List<string>> optionspaths = new Dictionary<string, List<string>>
             { {"/x", new List<string>()},{"/i",new List<string>()},{"/xf",new List<string>()},{"/if",new List<string>()},
             {"/xd",new List<string>()},{"/id",new List<string>()},{"/paths",new List<string>()} };
         public static Dictionary<string, List<Regex>> optionsregex = new Dictionary<string, List<Regex>>
             { {"/xr", new List<Regex>()},{"/ir",new List<Regex>()},{"/xfr",new List<Regex>()},{"/ifr",new List<Regex>()},
             {"/xdr",new List<Regex>()},{"/idr",new List<Regex>()} };        
         public static Dictionary<string, Boolean> optionsbools = new Dictionary<string, bool> { { "/recurse", false }, { "/noerr", false }, {"/delete",false}, {"/xj", false}};
         public static long numOfDupes, dupeBytes, bytesrecovered, deletedfiles = 0;  // number of duplicate files found, bytes in duplicates, bytes recovered from deleting dupes, number of deleted dupes.
         public static long errors = 0;
         public static string delconfirm;        
 
         public static void Main(string[] args)
         {
             DateTime startTime = DateTime.Now;
             Console.WriteLine("Findup.exe v2.0 - By James Gentile - JamesRaymondGentile@gmail.com - 2012.");
             Console.WriteLine("Findup.exe matches sizes, then SHA512 hashes to identify duplicate files.");
             Console.WriteLine(" ");                        
             string optionskey = "/paths";        
             List<FileInfo> files = new List<FileInfo>();            
             int i = 0;            
 
             for (i = 0; i < args.Length; i++)
             {
                 string argitem = args[i].ToLower();
 
                 if ((System.String.Compare(argitem, "/help", true) == 0) || (System.String.Compare(argitem, "/h", true) == 0))
                 {
                     Console.WriteLine(" ");
                     Console.WriteLine("Options:  /help     - displays this help message.");
                     Console.WriteLine("          /recurse  - recurses through subdirectories when directories or file specifications (e.g. *.txt) are specified.");                    
                     Console.WriteLine("          /noerr    - discards error messages.");
                     Console.WriteLine("          /delete   - delete each duplicate file with confirmation.");                    
                     Console.WriteLine("          /x        - eXcludes if full file path starts with (or RegEx matches if /xr) one of the items following this switch until another switch is used.");
                     Console.WriteLine("          /i        - include if full file path starts with (or Regex matches if /ir) one of the items following this switch until another switch is used.");
                     Console.WriteLine("          /xd       - eXcludes all directories - minus drive/files - (using RegEx if /xdr) including subdirs following this switch until another switch is used.");
                     Console.WriteLine("          /id       - Include only directories - minus drive/files - (using RegEx if /idr) including subdirs following this switch until another switch is used.");
                     Console.WriteLine("          /xf       - eXcludes all files - minus drive/directories - (using RegEx if /xfr) following this switch until another switch is used.");
                     Console.WriteLine("          /if       - Include only files - minus drive/directories - (using RegEx if /ifr) following this switch until another switch is used.");
                     Console.WriteLine("          [r]       - Use regex for include/exclude by appending an 'r' to the option, e.g. -ir, -ifr, -idr, -xr, -xfr, -xdr.");
                     Console.WriteLine("          /paths    - not needed unless you want to specify files/dirs after an include/exclude without using another non-exclude/non-include option.");
                     Console.WriteLine("          /xj       - Exclude File and Directory Junctions.");
                     Console.WriteLine(" ");
                     Console.WriteLine("Examples: findup.exe c:\\finances /recurse /noerr /delete");
                     Console.WriteLine("                     - Find dupes in c:\\finance.");
                     Console.WriteLine("                     - recurse all subdirs of c:\\finance.");
                     Console.WriteLine("                     - suppress error messages.");
                     Console.WriteLine("                     - deletes duplicates after consent is given.");                    
                     Console.WriteLine("          findup.exe c:\\users\\alice\\plan.txt d:\\data /recurse /x d:\\data\\webpics");
                     Console.WriteLine("                     - Find dupes in c:\\users\\alice\\plan.txt, d:\\data");
                     Console.WriteLine("                     - recurse subdirs in d:\\data.");
                     Console.WriteLine("                     - exclude any files in d:\\data\\webpics and subdirs.");
                     Console.WriteLine("          findup.exe c:\\data *.txt c:\\reports\\quarter.doc /xfr \"(jim)\"");
                     Console.WriteLine("                     - Find dupes in c:\\data, *.txt in current directory and c:\\reports\\quarter.doc");
                     Console.WriteLine("                     - exclude any file with 'jim' in the name as specified by the Regex item \"(jim)\"");
                     Console.WriteLine("          findup.exe c:\\data *.txt c:\\reports\\*quarter.doc /xr \"[bf]\" /ir \"(smith)\"");
                     Console.WriteLine("                     - Find dupes in c:\\data, *.txt in current directory and c:\\reports\\*quarter.doc");
                     Console.WriteLine("                     - Include only files with 'smith' and exclude any file with letters b or f in the path name as specified by the Regex items \"[bf]\",\"(smith)\"");
                     Console.WriteLine("Note:     Exclude takes precedence over Include.");
                     return;
                 }
                 if (optionsbools.ContainsKey(argitem))
                 {
                     optionsbools[argitem] = true;
                     optionskey = "/paths";
                     continue;
                 }                
                 if (optionspaths.ContainsKey(argitem) || optionsregex.ContainsKey(argitem))
                 {
                     optionskey = argitem;
                     continue;
                 }                
                 if (optionspaths.ContainsKey(optionskey))                
                     optionspaths[optionskey].Add(args[i]);                                    
                 else 
                 {
                     try {
                         Regex rgx = new Regex(args[i], RegexOptions.Compiled);
                         optionsregex[optionskey].Add(rgx);
                     }
                     catch (Exception e) {WriteErr("Regex compilation failed: " + e.Message);}
                 }
             }
             if (optionspaths["/paths"].Count == 0)
             {
                 WriteErr("No files, file specifications, or directories specified. Try findup.exe /help. Assuming current directory.");
                 optionspaths["/paths"].Add(".");
             }
             Console.Write("Getting file info and sorting file list...");
             getFiles(optionspaths["/paths"], "*.*", files, optionsbools["/recurse"], optionsbools["/xj"]);
                          
             if (files.Count < 2)
             {
                 WriteErr("\nFindup.exe needs at least 2 files to compare. Try findup.exe /help");
                 Console.WriteLine("\n");
                 return;
             }
 
             files.Sort(new FileLengthComparer());
             Console.WriteLine("Completed!");
             
             Console.WriteLine("Building dictionary of file sizes, SHA512 hashes and full paths...");
             
             for (i = 0; i < (files.Count - 1); i++)
             {
                 if (files[i].Length != files[i + 1].Length) continue;
 
                 var breakout = false;
                 var HashFile = new Dictionary<string, List<FileInfo>>();
                 long filesize = files[i].Length;
 
                 while (true)
                 {
 
                     try
                     {
                         var _SHA512 = SHA512.Create();
                         using (var fstream = File.OpenRead(files[i].FullName))
                         {
                             _SHA512.ComputeHash(fstream);
                         }
 
                         string SHA512string = Hash2String(_SHA512.Hash);
 
                         if (!HashFile.ContainsKey(SHA512string))
                             HashFile[SHA512string] = new List<FileInfo>() { };
                         HashFile[SHA512string].Add(files[i]);
                     }
                     catch (Exception e) { WriteErr("Hash error: " + e.Message); }
 
                     if (breakout == true) { break; }
                     i++;
                     if (i == (files.Count - 1)) { breakout = true; continue; }
                     breakout = (files[i].Length != files[i + 1].Length);
                 }
 
 
                 foreach (var HG in HashFile)
                 {
                     if (HG.Value.Count > 1)     // display filenames if number of files in SHA512 Hash Group > 1
                     {
                         Console.WriteLine("{0:N0} Duplicate files. {1:N0} Bytes each. {2:N0} Bytes total : ", HG.Value.Count, filesize, filesize * HG.Value.Count);
                         foreach (var finfo in HG.Value)
                         {
                             Console.WriteLine(finfo.FullName);
                             numOfDupes++;
                             dupeBytes += filesize;
                             if (optionsbools["/delete"])
                                 if (DeleteDupe(finfo)) { bytesrecovered += filesize; deletedfiles++; }
                         }
                     }
                 }
             }
                       
 
             Console.WriteLine("\n ");
             Console.WriteLine("Files checked      : {0:N0}", files.Count);              // display statistics and return to OS.
             Console.WriteLine("Duplicate files    : {0:N0}", numOfDupes);
             Console.WriteLine("Duplicate bytes    : {0:N0}", dupeBytes);
             Console.WriteLine("Deleted duplicates : {0:N0}", deletedfiles);
             Console.WriteLine("Bytes recovered    : {0:N0}", bytesrecovered);
             Console.WriteLine("Errors             : {0:N0}", errors);
             Console.WriteLine("Execution time     : " + (DateTime.Now - startTime));
         }               
 
         private static void WriteErr(string Str)
         {
             errors++;
             if (!optionsbools["/noerr"])
                 Console.WriteLine(Str);
         }
         private static string Hash2String(Byte[] hasharray)
         {
             string SHA512string = "";
             foreach (var c in hasharray)
             {
                 SHA512string += String.Format("{0:x2}", c);
             }
             return SHA512string;
         }
 
         private static Boolean DeleteDupe(FileInfo Filenfo)
         {
             Console.Write("Delete this file <y/N> <ENTER>?");
             delconfirm = Console.ReadLine();
             if ((delconfirm[0] == 'Y') || (delconfirm[0] == 'y'))
             {
                 try
                 {                    
                     Filenfo.Delete();
                     Console.WriteLine("File Successfully deleted.");                                        
                     return true;
                 }
                 catch (Exception e) { Console.WriteLine("File could not be deleted: " + e.Message); }
             }
             return false;
         }
 
 
         private static Boolean CheckNames(string fullname)
         {
             var filePart = Path.GetFileName(fullname);                                                              // get file name only (e.g. "d:\temp\data.txt" -> "data.txt")
             var dirPart = Path.GetDirectoryName(fullname).Substring(fullname.IndexOf(Path.VolumeSeparatorChar)+2);  // remove drive & file  (e.g. "d:\temp\data.txt" -> "temp")
 
             if (CheckNamesWorker(fullname, "/x", "/xr", true))
                 return false;
             if (CheckNamesWorker(filePart, "/xf", "/xfr", true))           
                 return false;
             if (CheckNamesWorker(dirPart, "/xd", "/xdr", true))            
                 return false;            
             if (CheckNamesWorker(fullname, "/i", "/ir", false))
                 return false;
             if (CheckNamesWorker(filePart, "/if", "/ifr", false))
                 return false;
             if (CheckNamesWorker(dirPart, "/id", "/idr", false))
                 return false;
             return true;
         }
         
         private static Boolean CheckNamesWorker(string filestr, string pathskey, string rgxkey, Boolean CheckType)
         {            
             foreach (var filepath in optionspaths[pathskey])
             {
                 if (filestr.ToLower().StartsWith(filepath.ToLower()) == CheckType)
                     return true;                    
             }           
             foreach (var rgx in optionsregex[rgxkey])
             {
                 if (rgx.IsMatch(filestr) == CheckType)
                     return true;
             }            
             return false;
         }        
                 
         private static void getFiles(List<string> pathRec, string searchPattern, List<FileInfo> returnList, Boolean recursiveFlag = true, Boolean xj = true)
         {
             foreach (string d in pathRec) { getFiles(d, searchPattern, returnList, recursiveFlag, xj); }         
         }
 
         private static void getFiles(string[] pathRec, string searchPattern, List<FileInfo> returnList, Boolean recursiveFlag = true, Boolean xj = true)
         {
             foreach (string d in pathRec) { getFiles(d, searchPattern, returnList, recursiveFlag, xj); }            
         }
 
         private static void getFiles(string pathRec, string searchPattern, List<FileInfo> returnList, Boolean recursiveFlag = true, Boolean xj = true)
         {
 
             string dirPart;
             string filePart;
 
             if (File.Exists(pathRec))
             {
                 try
                 {
                     FileInfo addf = (new FileInfo(pathRec));
                     if (((addf.Attributes & FileAttributes.ReparsePoint) == 0) || !xj)
                         if (CheckNames(addf.FullName))                        
                             returnList.Add(addf);
                 }
                 catch (Exception e) { WriteErr("Add file error: " + e.Message); }                
             }
             else if (Directory.Exists(pathRec))
             {
                 try
                 {
                     DirectoryInfo Dir = new DirectoryInfo(pathRec);
                     if (((Dir.Attributes & FileAttributes.ReparsePoint) == 0) || !xj)
                         foreach (FileInfo addf in (Dir.GetFiles(searchPattern)))
                         {                        
                             if (((addf.Attributes & FileAttributes.ReparsePoint) == 0) || !xj)
                                 if (CheckNames(addf.FullName))                        
                                     returnList.Add(addf);
                         }
                 }
                 catch (Exception e) { WriteErr("Add files from Directory error: " + e.Message); }
 
                 if (recursiveFlag)
                 {
                     try { getFiles((Directory.GetDirectories(pathRec)), searchPattern, returnList, recursiveFlag, xj); }
                     catch (Exception e) { WriteErr("Add Directory error: " + e.Message); }
                 }                
             }
             else
             {
                 try
                 {
                     filePart = Path.GetFileName(pathRec);
                     dirPart = Path.GetDirectoryName(pathRec);
                 }
                 catch (Exception e)
                 {
                     WriteErr("Parse error on: " + pathRec);
                     WriteErr(@"Make sure you don't end a directory with a \ when using quotes. The console arg parser doesn't accept that.");
                     WriteErr("Exception: " + e.Message);
                     return;
                 }
 
                 if (filePart.IndexOfAny(new char[] {'?','*'}) >= 0)
                 {
                     if ((dirPart == null) || (dirPart == ""))
                         dirPart = Directory.GetCurrentDirectory();
                     if (Directory.Exists(dirPart))
                     {
                         getFiles(dirPart, filePart, returnList, recursiveFlag, xj);
                         return;
                     }
                 }
                 WriteErr("Invalid file path, directory path, file specification, or program option specified: " + pathRec);                                                        
             }            
         }
     }
 }
`

