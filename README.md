# flip_cdetype

Fix blank covers on sideloaded Kindle books by marking them as personal documents.

## The problem

On older Kindles, a sideloaded book got its cover from a thumbnail image copied into the hidden `system/thumbnails` folder. The 2026 Kindles block access to that folder, both from file managers like Amazon's USB File Manager and from Calibre. Books from sources like Standard Ebooks therefore show a generic cover with only the title and author.

## Why this fixes it

Every Kindle book file carries a content-type tag (`cdetype`, stored in EXTH record 501):

- **EBOK** ("store book"). The Kindle expects a thumbnail from Amazon or from `system/thumbnails`. If neither is available, the book gets a blank cover. Standard Ebooks and Calibre use this tag by default.
- **PDOC** ("personal document"). The Kindle generates the cover itself from the image embedded in the book.

This script changes the tag from EBOK to PDOC and nothing else. The book's text, styling, images and all other metadata stay byte-for-byte identical. A typical file differs by just 3 bytes.

## Requirements

- Python 3 (no extra packages). On macOS, the first run of `python3` may prompt you to install the command-line developer tools.
- Works with `.azw3`, `.azw` and `.mobi` files. KFX files aren't supported. For KFX, use Calibre's KFX Output plugin with "Create book (EBOK)" unchecked.

## Usage

Convert a whole folder:

```
python3 flip_cdetype.py ~/Downloads/standard-ebooks
```

The converted copies are written to a `pdoc` folder inside that folder, and the originals are never modified.

Then load the books onto the Kindle:

1. Connect the Kindle and open Amazon's USB File Manager on a Mac, or File Explorer on Windows.
2. Delete any earlier copies of the same books from the Kindle's `documents` folder. Otherwise you'll end up with duplicates.
3. Copy the files from the `pdoc` folder into `documents`.
4. Eject the Kindle. The books appear under the **Documents** filter, and their covers should show.

### Other options

| Command | What it does |
| --- | --- |
| `python3 flip_cdetype.py book1.azw3 book2.azw3` | Convert specific files |
| `python3 flip_cdetype.py FOLDER -o ~/Desktop/kindle-ready` | Write output to a folder you choose |
| `python3 flip_cdetype.py FOLDER --check` | Report each file's current tag without changing anything |
| `python3 flip_cdetype.py pdoc --type EBOK` | Undo: set the tag back to EBOK (restores the original file exactly) |

Files that are already PDOC are copied over unchanged. When you download new books, put them in the same folder and run the script again.

## Trade-offs

- PDOC books are listed under **Documents** instead of **Books** in the Kindle library. Collections, search and sorting still work.
- On the Kindle, the book's menu shows **Permanently delete** instead of **Remove download**.
- Store-only features such as X-Ray and Word Wise aren't available. Sideloaded books never had them anyway, because those features require a matching Amazon store listing.
- Books loaded over USB don't sync reading position across devices, whatever their tag.

## Fallback

If a book still shows a blank cover after conversion, the alternative is to convert it to KFX as a personal document. In Calibre, install the KFX Output plugin and Amazon's Kindle Previewer, convert the EPUB to KFX with "Create book (EBOK) instead of personal document (PDOC)" unchecked, and send it to the device.

## Technical notes

- If a file already has an EXTH 501 record, the script overwrites its 4-byte value in place, so no lengths or offsets change. If the record is missing, the script adds it and recalculates the affected header offsets and record table.
- For older combined MOBI/KF8 files, both headers are updated. The script finds the second header through EXTH record 121.
- Each output file is re-read and checked before it's written. Anything that isn't a valid Kindle file is reported as `FAILED` and skipped.
