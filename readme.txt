Introduction

A few years ago I did some research on how Windows can boot a wim (wimboot). Link to one of the original threads: http://www.msfn.org/board/topic/145209-the-smallest-possible-size-of-bootsdi/ For instance WinPE based on Vista and later started using the wim format for its ramdisk booting. Basically what happens is that bootmgr locates boot.sdi, which contains a small NTFS partition, and then mounts the wim (boot.wim) onto the small NTFS partition. Then execution is transferred to winload.exe which resides inside boot.wim which then loads drivers capable of handling the environment (wim based ramdisk), and then transfer control further to the kernel. 

Details

What I found is that WinPE does not care about the NTFS partition at the bottom being in perfect healthy shape. I was driven by the goal of reducing the size of the timy partition as much as possible. So I started out with some lame trial and error. One of the things that really annoyed me, and really made no sense, was the fact that the original partition was 3 MB in size and contained free space. After removing all the free space I hit the first barrier, when the size of the partition was equivalent to the size of the NTFS system files, whith the $LogFile? being the biggest of them all. After some fiddling, I was redirected to ntfsfloppy image that Mark Russinovich made many years ago. After reading his description of the process (debugging with SoftIce?), that ultimately lead to a severely shrinked $LogFile?, I realized that this could probably be done with the right tools. After some reverse engineering and debugging in Olly, I realized that untfs.dll is where the size of $LogFile? is set. Notes on this can be collected in the above referenced thread. I found that the lowest possible size was at 256 KB. Not exactly true, as I could patch my may all down to 64 KB, but that did not work with the NTFS drivers, so 256 KB seemed like the smallest size working with happy NTFS drivers. Ok, so when reassembling the partition with the 256 KB $LogFile?, its size was reduced below 1 MB, which started looking fine. By coincidence I tried overlapping some of the NTFS metafiles, and WinPE did not seem to complain. So some trial and error later, I had found that the combination of overlapping systemfiles that gave the lowest partition size, while still making WinPE boot fine on it, gave a partition size of 293 KB (300 544 bytes). With overlapping systemfiles I mean systemfiles occupying the same sectors on disk. The point is that WinPE only cared for some of the systemfile's presence, and not its content (or only parts of its content). So on certain sectors, up to 3 systemfiles had a reference to. And some systemfiles where fully contained within the $LogFile?. When I got this far, no tools would really do the work for me, so I ended up writing a large part of this version entirely by hand using a hex editor as the only tool. Files available in the download section. 

The layout of the final partition (systemfile and sector number): 
•$Boot 0 
•$MFT 1-65 
•Root directory 66-73 
•$LogFile? 74-586 
•$MFTMirr 136-143 
•$UpCase? 136 
•$Secure:$SDS 137-392 
•$Bitmap 138 
•$AttrDef? 139-143 


As can be seen, the first four entries sets the boundary of the tiny partition. 

The rest of the systemfiles, not mentioned above, are resident (fully contained within $MFT). 

Please note that the volume is invalid, so no tool will help you much editing it, except a hex editor. Also note that some of the systemfiles are invalid too, and only works in this particular scenario. 

Anyways, the final boot.sdi was about a tenth of the original size, which I was happy with. Because of its small size it was possible to put it onto floppy images that utilized multiboot scenarios. For instance NTBOOT makes use of this boot.sdi; http://chenall.net/post/ntboot/ 

In Windows 8 the criteria changed and the requirements for boot.sdi is now to hold a valid partition. The exact details are not yet sorted completely, but I have a 960 KB version that works, and possibly could be further reduced.. 
