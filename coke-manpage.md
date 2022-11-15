# coke - CONVERT file with any tool and KEEP parent directory, basename and filesystem metadata for output file.

## Syntax
- Two basic forms, 3 or 5 params.
- Parameters are strictly positional.

## Coke mode I is useful when the conversion already ran elsewhere
- e.g. in CLI earlier, or in a GUI app, which can't be run in coke mode II.
- Copies your desired `metadata` from `srcFile` to `dstFile`

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


## Coke mode II makes ANY CLI conversion tool totally filesystem metadata friendly/preserving

- And reduces the hassle of file naming/moving to a minimum.
- In a single go coke mode II runs your conversion, preserves parent folder, filename, and all kind of filesystem metadata, and performs renaming of `src` and/or `dst`.


```
	II) coke 'command line with literal "$src" and $dst" somewhere' <srcSuffix> <dstSuffix> <metadataFlags> <srcFile>
```

1. is the complete `cmdLine` for the respective 'conversion tool with literal placeholders "$src" and "$dst" somewhere'.
  - Use syntax exactly as stated:
     - Whole command line wrapped in single quotes.
     - Variable names prefixed with dollar symbol within double quotes so that whitespace filenames work too.
  - Any further quotes within the outer quotes are at your own risk.
2. is the optional `srcSuffix`, e.g. " obsolete"
  - Cannot be set to the special value "_converted" which is reserved as the hardcoded fallback for dstSuffix. Wouldn't make sense as srcSuffix anyhow.
  - Set as "" empty to keep your src name unchanged.
3. is the optional `dstSuffix`, e.g. " converted"
  - Inserted at the end of the filename just before the dot and the file extension.
  - **Important**: See special values for `srcSuffix` and `dstSuffix` and their interplay and outcomes in the next section!
4. are the `metaFlags`, see above at I.
5. is the `srcFile` to be used in your conversion whose output file will keep parent directory, basename and FS metadata.


## How you set the suffixes determines the outcome!

- A suffix which shall remain unchanged uses empty quotes "" at the correct positional parameter.
- Also beware of the interplay of `srfSuffix` and `dstSuffix`!

1. Set only `dstSuffix`, e.g. " converted", then:
  - src remains unchanged.
  - dst gets the filename of src plus the dstSuffix inserted before the file extension.
  - At the same parent directory and with all the FS metadata from the src file.
2. Set only `srcSuffix` e.g. " obsolete", then:
  - During conversion coke will set `dstSuffix` to the hardcoded value "_converted".
  - After conversion `src` gets renamed with `srcSuffix`, e.g. " original", and `dst` now has `srcFile` 's name and metadata.
3. Set both suffixes if you want both renamed after the conversion.
  - But never set them to the same suffix as this would result in a name conflict!
  - The program aborts if you set the suffixes to a same non-null value, but beware nonetheless!
4. Set both suffixes as empty "" "" to mimick destructive in-place conversion for tools that natively don't support this.
  - Use with care! The original is gone after this!
  - During conversion `coke` will set `dstSuffix` to the hardcoded value "_converted".
  - After conversion `coke` copies all FS metadata from `src` to `dst`.
  - Then it deletes `src`. Then it renames `dst` to `src`.
  - You are left with only one file: The converted file which now has the src's filepath and FS metadata.
  - See example 4


## Examples

### 1) Drop audio track from video file

	coke 'ffmpeg -i \$src -c copy -an "$dst"' "" "-audio-only" "cmf" ~/videos/ant.mp4

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
