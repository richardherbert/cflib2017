---
layout: udf
title:  ImageSize
date:   2010-03-20T17:03:39.000Z
library: UtilityLib
argString: "filename[, mimetype]"
author: Peter Crowley
authorEmail: pcrowley@webzone.ie
version: 2
cfVersion: CF6
shortDescription: Returns width and height of images based on image type.
tagBased: false
description: |
 Return X resolution in pixels and Y resolution in pixels of an Image. Supports JPGs and GIFs.

returnValue: Returns a struct containing height and width information, or an error string.

example: |
 <CFSCRIPT>
 kImageSize = ImageSize("c:\inetpub\wwwroot\sample.jpg");
 </CFSCRIPT>
 
 <CFOUTPUT>
 #kImageSize.ImageWidth#<BR/>
 #kImageSize.ImageHeight#
 </CFOUTPUT>

args:
 - name: filename
   desc: Absolute or relative path to file.
   req: true
 - name: mimetype
   desc: Minetype for the file.
   req: false


javaDoc: |
 /**
  * Returns width and height of images based on image type.
  * v2 fix by John Bliss
  * 
  * @param filename      Absolute or relative path to file. (Required)
  * @param mimetype      Minetype for the file. (Optional)
  * @return Returns a struct containing height and width information, or an error string. 
  * @author Peter Crowley (pcrowley@webzone.ie) 
  * @version 2, March 20, 2010 
  */

code: |
 function ImageSize(filename) {
     // Jpeg variables
     var nFileLength=0; var nBlockLength=0; var nMarker=0;
     var nSOI = 65496; // Start of Image (FFD8)
     var nEOI = 65497; // End of Image (FFD9)
     var nSOF = 65472; // Start of frame nMarker (FFC0)
     var nSOF1 = 65473; // Start of frame extended sequential mode (FFC1)
     var nSOF2 = 65474; // Start of frame progressive mode (FFC2)
     var nSOF3 = 65475; // Start of frame lossless mode (FFC3)
     var nSOS = 65498; // Start of Scan (FFDA)
 
     
     var sImageType = "";
     var kCoords = structNew();
     var fInput = 0;
     var sByte=0;
     var sFullPath="";
     var sMimeType = "";
     
     if (Left(filename,1) IS "/" OR Left(filename,1) IS "\" OR MID(filename,2,1) IS ":")
         sFullPath=filename;
     else
         sFullPath=ExpandPath(filename);
 
     // Establish image type 
     if(arrayLen(arguments) gt 1) {     //optional mimetype
         sMimeType = arguments[2];
         if (LCase(ListFirst(sMimeType,"/")) IS NOT "image") return "Wrong mime type";
         if (ListLen(sMimeType,"/") NEQ 2) return "Invalid mime type";
         sImageType=LCase(ListLast(sMimeType,"/"));
     } else { // work off file extension
         if (ListLen(filename,".") LT 2) return "Unknown image type";
         sImageType=LCase(ListLast(filename,"."));
     }
 
     if(not fileExists(sFullPath)) return "File does not exist.";
     
     //make a fileInputStream object to read the file into
     fInput = createObject("java","java.io.RandomAccessFile").init(sFullPath,"r");
     
     // Get X,Y resolution sizes for each image type supported
     switch (sImageType) {
     case "jpg": case "jpeg":
         do {
             nMarker = fInput.readUnsignedShort();
 
             if (nMarker NEQ nSOI AND nMarker NEQ nEOI AND nMarker NEQ nSOS) {
 
                 nBlockLength = fInput.readUnsignedShort();
 
                 if (nMarker EQ nSOF OR nMarker EQ nSOF1 OR nMarker EQ nSOF2 OR nMarker EQ nSOF3) { // Start of frame
                     fInput.readUnsignedByte(); // skip sample precision in bits
                     kCoords.ImageHeight = fInput.readUnsignedShort();
                     kCoords.ImageWidth = fInput.readUnsignedShort();
                     fInput.close();
                     return kCoords;
                 } else {
                     fInput.skipBytes(JavaCast("int",nBlockLength-2));
                 }
             }
         } while (BitSHRN(nMarker,8) EQ 255 AND nMarker NEQ nEOI);
         break;
     case "gif":
         fInput.skipBytes(6);
         
         sByte = fInput.readUnsignedByte();
         kCoords.ImageWidth = fInput.readUnsignedByte() * 256 + sByte;
 
         sByte = fInput.readUnsignedByte();
         kCoords.ImageHeight = fInput.readUnsignedByte() * 256 + sByte;
 
         fInput.close();
         return kCoords;
     default:
         break;
     }
     //close out this entry
     fInput.close();
     return "Unhandled image type";
 }

oldId: 1019
---

