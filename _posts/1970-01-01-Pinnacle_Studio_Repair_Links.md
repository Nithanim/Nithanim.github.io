---
layout: posts
title:  "Pinnacle Studio: Repair broken links"
date:   1970-01-01 01:00:00 +0100
author: Nithanim
categories: misc 
---

Someone had the problem, that the linkings of the mediafiles were messed up completely resulting in Pinnacle Studio 16 (PS) using (only) partly the wrong video files.
The source of the problem was that the capturing device numbers the files consecutively starting from 00000.
Since two cards were used they were saved in two folders on the hdd, one for each card.
When they were moved, the link broke and PS asked to relink the files.
Since the files of folder "two" had the same filenames as the of the first one, PS thought that it had already found all required files and then the mess was perfect.
As a sidenote, I used PS 17 while fixing this issue.

## Before we start
MAKE A BACKUP!!! And I am not responsible for any damages to anything. (As always)

## The fiddle
The *.Movie.axp is a simple XML-file which can be opened with a normal text editor (I recommend Notepad++; Select XML in the "Languages" menu).
There are generally two places where PS stores its linkings - in the Library and in this file.
It is important to note that the linkings stored in the Library are preferred, thus if you change something in the file you need to delete the files from the Library (select from Library only, NOT delete Files!).

### <Item>
Basically, a lot of things in this File is a `<Item>` but we are interested in the items holding a clip.
They can be distinguished by an `<Object ClassNBame="CNGTrackClip">` from the others.
If you didn't change the the name/title/description or whatever it is called, it still contains the original file name somewhere inbetween these tags.
Such a clip could look like this:
```
<Item MarkIn="4029ae147ae147ae(12.84)" MarkOut="4036cccccccccccc(22.8)" RecIn="4045eb851eb851eb(43.84)">
  <EntryData>
    <Object ClassName="CNGTrackClip">
      <CNGObject ExpectedChildListSize="2" />
      <CNGTrackClip>
        <Clip>
          <Object ClassName="CNGStreamSource">
            <CNGObject CreatorID="3729AC0A-27CB-48B5-B625-84B0362B3C88">
              <Name>00020.MTS</Name>
            </CNGObject>
            <CNGMediaSource>
              <Routing>
                <Category AllowedStreamMaskV2="1" StreamMask="1" ID="0" />
                <Category AllowedStreamMaskV2="1" StreamMask="1" ID="1" />
              </Routing>
              <Media Reference="4380" />
            </CNGMediaSource>
            <CNGStreamSource ClipIn="0(0)" ClipOut="40487ae147ae147b(48.96)" />
          </Object>
        </Clip>
        <AudioFader>
          <Object ClassName="CNGFaderKeys" StaticVolume="0" />
        </AudioFader>
      </CNGTrackClip>
    </Object>
  </EntryData>
</Item>
```
Note the `<Object ClassName="CNGTrackClip">` and `<CNGObject CreatorID="3729AC0A-27CB-48B5-B625-84B0362B3C88">`, where the latter plays an essential role in our repairing process.

### <Directory>
Another essential part is at the end of our File in the <MediaArchive>.
If you are using Notepad++, you can hide (not delete!) the `<BinaryArchive>` with a click on the on the left side, because we are not interested in it and slows down our editor (at least Notepad++).
```
<Directory Path="e:/path/">
  <File FileID="4379" GUID="35F6100A-D404-4203-BB4B-CAA33303478A" FileCRC="5B56BFB8-D0B2-F3BA-9EFC-90D8AF26EFC8">00129.mts</File>
  <File FileID="4380" GUID="3729AC0A-27CB-48B5-B625-84B0362B3C88" FileCRC="CA206931-9565-E11F-405F-DCF670D057AD">00020.mts</File>
  <File FileID="4382" GUID="BB07B20C-1E47-43D7-B54B-50F8056C9518" FileCRC="DA12C15C-94D0-F129-C67F-C382C974FB95">00207.mts</File>
</Directory>
```
A `<Directory>` simply denotes a source directory.
Note that the Path uses "/" instead of "\" as path separators!

A file has several attributes (FileID, GUID and FileCRC) and holds the filename in it.
The FileID seems to be a four-digit-number and unique in the file, so it is a good idea to keep it this way.
The GUID is the same as the CreatorID of the Item in our timeline, so if we want change the file it is pointing to, we need to change it to the right GUID of the file we want PS to use instead.
I am neither sure how it is generated nor where it is used.

## Fixing

### Relink only
In the best case, a `<Item>` simply links to the wrong file, but the right one is listed in one of the `<Directory>`s. The only thing you need to do is to copy the `GUID` for the right file and replace the `CreatorID` of the `<Item>` and you are done.

### Media not listed
It gets trickier if you want your `<Item>` to hold a file that is not listed for any reason in any of your `<Directory>`s.
You need to create a new `<File>` in the right `<Directory>`.
As `FileID` it appears that you need to choose one that is not used in this file (maybe take the highest number + 1).
For the `GUID` I believe you can take pretty much anything (still follow the convention) that is unique, but It makes sense to choose something the generator may not create.
In terms of the `FileCRC` I simply filled it with 0's (again following the convention). I chose the following for me:
```
<File FileID="4460" GUID="NITHANIM-0000-0000-0000-000000000001" FileCRC="00000000-0000-0000-0000-000000000000">00191.mts</File>
```

## Final touch
To take effect, remember to close the project before attempting to modify the file.
Also, it is important to remove the Files in question (what you have changed) from the Library, otherwise YOU WON'T SEE ANY CHANGES because the Library is more important than the file.
The files are re-added to the Library every time you open the project so they need to be removed each and every time.
