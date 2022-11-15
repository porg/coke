# Coke
**CONVERT** file with any tool and **KEEP** parent directory, basename and filesystem metadata for output file.
- See [coke-manpage.md](https://github.com/porg/coke/blob/main/coke-manpage.md) for full syntax

## The basic problem that Coke solves
- **Most Unix command line tools don't care much about your (Mac) filesystem metadata** like creation date, modification date, Spotlight Comments, Finder Tags.
- When you feed them a file as input which is rich in filesystem metadata the output file almost never has that filesystem metadata but is blank in that regard.
- **Coke in mode II solves this problem right at the source** by being a wrapper around your favorite command line tool and takes care that:
  - the metadata you want to preserve carries over,
  - you have only a minimal effort for naming, renaming, moving, cmdLine re-typing,
  - it automates all that tedious stuff for you
- **Coke in mode I offers a fix in retrospect** by copying over metadata from an original to a converted file if the conversion has run without coke's supervision, be it a GUI app or a CLI which ran before you were using coke.


## Coke mode I if the conversion has already ran but you want to carry over metadata in retrospect
- Copies your desired `metadata` from `srcFile` to `dstFile`
- When the conversation already ran in CLI earlier, or in a GUI app.

```
I) coke <metadataFlags> <srcFile> <dstFile>
```

- `metadataFlags` can be: `"" | [c][m][e][E][f]`
  - `""` Supplying an empty argument uses the defaults: `cf`
  - `c`  Creation Date
  - `m`  Modification Date
  - `e`  File Extension Visibility (as stored in FS),
  - `E`  File Extension gets hidden on `dst`. Note: Omit `e` and `E` from `metadataFlags` and the file extension will be visible by default for new files in macOS Finder.
  - `f`  Filesystem Metadata (Finder Tags, Spotlight Comment, etc)


## Coke mode II transforms ANY CLI conversion tool into a filesystem metadata friendly solution

- And reduces the hassle of file naming/moving to a minimum.
- In a single go it runs your conversion, preserves parent folder, filename, and all kind of filesystem metadata, and performs renaming of `src` and/or `dst`.


```
II) coke 'command line with literal "$src" and $dst" somewhere' <srcSuffix> <dstSuffix> <metadataFlags> <srcFile>
```

1. is the complete `cmdLine` for the respective 'conversion tool with literal placeholders "$src" and "$dst" somewhere'.
2. is the optional `srcSuffix`, e.g. " obsolete"
3. is the optional `dstSuffix`, e.g. " converted"
  - **Important**: See special values for `srcSuffix` and `dstSuffix` and their interplay and outcomes in [coke-manpage.md](https://github.com/porg/coke/blob/main/coke-manpage.md#how-you-set-the-suffixes-determines-the-outcome)
4. are the `metaFlags`.
5. is the `srcFile` to be used in your conversion whose output file will keep parent directory, basename and FS metadata.


## Examples

### 1) Drop audio track from video file

	coke 'ffmpeg -i "$src" -c copy -an "$dst"' "" "-audio-only" "cmf" ~/videos/ant.mp4

  - `ant.mp4`             Original file, untouched, not renamed.
  - `ant-audio-only.mp4`  Converted file, creation and modification timestamps and FS metadata restored from original.


### 2) Update copyright note and keep old file just for legal/archival purposes

	coke 'ffmpeg -i "$src" -c copy -metadata copyright="New copyright notice as of 2023: ..." "$dst"' " - Copyright notice as used 2010-2022" "" "cmE"  ~/videos/bee.mp4

  - `bee - Copyright as used 2010-2022.mp4`   Original file, untouched, renamed.
  - `bee.mp4`   Converted file with updated copyright notice in video file header, using old canonical filename, creation and modification timestamps ("cm") from original, file extension set to be hidden in Finder ("E") regardless how original did it. You could still manually `touch bee.mp4` now to indicate "something significantly enough changed today/now". Now can distribute this updated file to various places.


### 3) Transcode to newer codec but stick with same container format - Then distinguish old and new clearly

	coke 'ffmpeg -i "$src" -c:v libx265 -crf 26 -preset fast -c:a aac -b:a 128k "$dst"' " H264" " H265" "" ~/movies/cat.mp4

  - `cat H264.mp4`  Original file, changed name for clarity.
  - `cat H265.mp4`  Converted file, changed name, default metadata restored from original (because argument n°4 is "" which means to use the default `metadataFlags`). You may `touch cat H265.mp4` now to indicate in the modification date when the format conversion took place. But content wise (not quality wise) they are the same, hence keep the creation date and also the filesystem metadata (e.g. Finder tags, xattr)


### 4) Destructive In-Place Editing: Update watermark in file

	coke 'graphictool "$src" --superimpose ~/logo.png --bottom-center "$dst"' "" "" "" ~/graphics/dog.png

  - `dog.png`   Updated file with name and FS metadata of original. Original was discarded. Use this mode of operation with care!


### 5) Convert video to other container format without re-encoding

	coke 'ffmpeg -i "$src" -c copy "$dst"' "" "*.webm" "" ~/videos/fox.mkv


Note: As `dstSuffix` was in the special form "*.webm" for `dst` the filename is exactly as that of `src` but the file extension including the dot get changed to what comes after the asterisk, e.g. ".webm". Handy for some conversion utilities which interpret the file extension as the target file format.

  - `fox.mkv`   Original file, untouched, not renamed.
  - `fox.webm`  Converted file, timestamps and FS metadata restored from original.
  
  
## Installation
1. Move `coke` and `coke-manpage.md` into your executable directory, e.g. `~/bin/`
2. Make coke executable like this: `chmod +x ~/bin/coke`
3. coke is ready for use!

## Dependencies
- Dependencies:
  - `GetFileInfo` and `SetFile` from Apple Xcode / Developer Tools is used for getting and setting the `metadataFlags` `cmeE`
  - [osxmetadata](https://github.com/RhetTbull/osxmetadata) is used for copying the `metadataFlag` `f`
- Optional:
  - [bat](https://github.com/sharkdp/bat) displays the manpage (which is in Markdown syntax) in a nice and colored formatting
  - If not available falls back to `cat` which is available on any Unix system
  
## Operating Systems — Made for macOS, but potentially also useful on other Unix systems
- Creation Date, Spotlight Comment, Finder Tags are only available on macOS.
- But in general this provides quite a lot of functionality which could be also handy on Unix or Linux.
- It would only need minor adaptations for which tools to get/set specific metadata flags, but the general structure and logic is already laid out.
