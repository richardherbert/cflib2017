---
layout: udf
title:  ZipFileNew
date:   2004-01-19T17:16:28.000Z
library: FileSysLib
argString: "zipPath, toZip[, relativeFrom]"
author: Nathan Dintenfass
authorEmail: nathan@changemedia.com
version: 1
cfVersion: CF6
shortDescription: Create a zip file of a directory or just a file.
tagBased: false
description: |
 Pass a path to a directory or file and where you want to write the zip file, and POOF!: Zip file.
 
 Updated 1/19/04.  Added a third optional argument that allows for easy creation of relative paths in the zip file.

returnValue: Returns nothing.

example: |
 <cfscript>
     //make a zip file the /WEB-INF/cftags directory using the full path in the zip file
 //    ZipFileNew(expandPath("cftags.zip"),expandPath("/WEB-INF/cftags/"));
 
     //make a zip file the /WEB-INF/cftags directory using the path /cftags as the top level directory in the zip file
 //    ZipFileNew(expandPath("cftags_relative.zip"),expandPath("/WEB-INF/cftags/"),expandPath("/WEB-INF/"));
 
 </cfscript>

args:
 - name: zipPath
   desc: File name of the zip to create.
   req: true
 - name: toZip
   desc: Folder or full path to file to add to zip.
   req: true
 - name: relativeFrom
   desc: Some or all of the toZip path, from which the entries in the zip file will be relative
   req: false


javaDoc: |
 /**
  * Create a zip file of a directory or just a file.
  * 
  * @param zipPath      File name of the zip to create. (Required)
  * @param toZip      Folder or full path to file to add to zip. (Required)
  * @param relativeFrom      Some or all of the toZip path, from which the entries in the zip file will be relative (Optional)
  * @return Returns nothing. 
  * @author Nathan Dintenfass (nathan@changemedia.com) 
  * @version 1.1, January 19, 2004 
  */

code: |
 function zipFileNew(zipPath,toZip){
     //make a fileOutputStream object to put the ZipOutputStream into
     var output = createObject("java","java.io.FileOutputStream").init(zipPath);
     //make a ZipOutputStream object to create the zip file
     var zipOutput = createObject("java","java.util.zip.ZipOutputStream").init(output);
     //make a byte array to use when creating the zip
     //yes, this is a bit of hack, but it works
     var byteArray = repeatString(" ",1024).getBytes();
     //we'll need to create an inputStream below for writing out to the zip file
     var input = "";
     //we'll be making zipEntries below, so make a variable to hold them
     var zipEntry = "";
     var zipEntryPath = "";
     //we'll use this while reading each file
     var len = 0;
     //a var for looping below
     var ii = 1;
     //a an array of the files we'll put into the zip
     var fileArray = arrayNew(1);
     //an array of directories we need to traverse to find files below whatever is passed in
     var directoriesToTraverse = arrayNew(1);
     //a var to use when looping the directories to hold the contents of each one
     var directoryContents = "";
     //make a fileObject we can use to traverse directories with
     var fileObject = createObject("java","java.io.File").init(toZip);
     //which part of the file path should be excluded in the zip?
     var relativeFrom = "";
     
     //if there is a 3rd argument, that is the relativeFrom value
     if(structCount(arguments) GT 2){
         relativeFrom = arguments[3];
     }
     
     //
     // first, we'll deal with traversing the directory tree below the path passed in, so we get all files under the directory
     // in reality, this should be a separate function that goes out and traverses a directory, but cflib.org does not allow for UDF's that rely on other UDF's!!
     //
     
     //if this is a directory, let's set it in the directories we need to traverse
     if(fileObject.isDirectory())
         arrayAppend(directoriesToTraverse,fileObject);
     //if it's not a directory, add it the array of files to zip
     else
         arrayAppend(fileArray,fileObject);    
     //now, loop through directories iteratively until there are none left
     while(arrayLen(directoriesToTraverse)){
         //grab the contents of the first directory we need to traverse
         directoryContents = directoriesToTraverse[1].listFiles();
         //loop through the contents of this directory
         for(ii = 1; ii LTE arrayLen(directoryContents); ii = ii + 1){            
             //if it's a directory, add it to those we need to traverse
             if(directoryContents[ii].isDirectory())
                 arrayAppend(directoriesToTraverse,directoryContents[ii]);    
             //if it's not a directory, add it to the array of files we want to add
             else
                 arrayAppend(fileArray,directoryContents[ii]);    
         }
         //now kill the first member of the directoriesToTraverse to clear out the one we just did
         arrayDeleteAt(directoriesToTraverse,1);
     } 
     
     //
     // And now, on to the zip file
     //
     
     //let's use the maximum compression
     zipOutput.setLevel(9);
     //loop over the array of files we are going to zip, adding each to the zipOutput
     for(ii = 1; ii LTE arrayLen(fileArray); ii = ii + 1){
         //make a fileInputStream object to read the file into
         input = createObject("java","java.io.FileInputStream").init(fileArray[ii].getPath());
         //make an entry for this file
         zipEntryPath = fileArray[ii].getPath();
         //if we are making the zip relative from a certain directory, exclude that from the zipEntryPath
         if(len(relativeFrom)){
             zipEntryPath = replace(zipEntryPath,relativeFrom,"");
         } 
         zipEntry = createObject("java","java.util.zip.ZipEntry").init(zipEntryPath);
         //put the entry into the zipOutput stream
         zipOutput.putNextEntry(zipEntry);
         // Transfer bytes from the file to the ZIP file
         len = input.read(byteArray);
         while (len GT 0) {
             zipOutput.write(byteArray, 0, len);
             len = input.read(byteArray);
         }
         //close out this entry
         zipOutput.closeEntry();
         input.close();
     }
     //close the zipOutput
     zipOutput.close();
     //return nothing
     return "";
 }

oldId: 744
---

