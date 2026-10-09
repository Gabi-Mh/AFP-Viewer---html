# AFP Viewer

A single-file, browser-based viewer for AFP (MO:DCA) documents. Open a file, browse its pages, inspect its structure, and export pages to PNG or PDF. There is nothing to install and nothing is uploaded: files are read and drawn locally in your browser.

> Positions, rules, colours, overlays and page segments come straight from the file. Letter shapes and widths depend on the fonts you have, because AFP font resources are not read yet (see [Limitations](#limitations)). Load matching fonts to get close to the original.

## Quick start

1. Open `afp-viewer.html` in a recent Chrome, Edge, Firefox or Safari. No build step, no dependencies, no network access.
2. Click **Open files**, or drop files anywhere on the window. Select the `.afp` file plus any overlay, page-segment or image files it needs.
3. Click **Fonts folder** once and choose this repository's `Fonts` folder. The fonts are remembered in that browser and load automatically on later visits.
4. No AFP file at hand? Click **Try a sample page**.

It is a static page, so any static web host works too.

## Features

- **Pages:** thumbnails, page navigation, zoom (fit width, fit page, 50-300%, plus/minus).
- **Text:** absolute and relative positioning, baseline increments, rotation in 90 degree steps, colours (standard, RGB, CMYK), underline, and drawn rules. Text encodings: EBCDIC (code page 037), ASCII and UTF-16, with auto-detect.
- **Overlays and page segments:** included from the same file or from resource files.
- **Images:** JPEG, PNG and GIF (and TIFF where your browser can decode it) inside object containers and IOCA image data. Unsupported objects are drawn as dashed outlines.
- **Fonts:** size, weight and italic are read from the file's font entries when present; a Fonts tab lets you override the font, size, bold and italic per AFP font.
- **Structure tab:** the structured fields of the current page with a hex dump of each.
- **Export:** current page as PNG, all pages as PDF, or print the current page.
- Light and dark theme follow the system setting.

## Fonts

A web page cannot read a folder that sits next to it, so the viewer asks you to pick the folder once and remembers it (browser IndexedDB, per browser profile).

- **Fonts folder** loads every `.ttf`, `.otf`, `.woff`, `.woff2` (and `.def`) file in the folder and its subfolders, and saves them. **Forget saved fonts** (Fonts tab) clears them.
- **Load font**, or dropping a font file on the window, adds a single font. This also selects it for all text in the **Font** menu. Loading the folder does not change the Font menu.
- Font files are grouped into families by file name. Style suffixes such as `-Bold`, `-Italic`, `-BoldItalic`, `Oblique` are recognised, so `LiberationSans-Regular.ttf` and `LiberationSans-Bold.ttf` form one family.

### How a font is chosen

For each AFP font, the first match wins:

1. The choice you made for that font in the **Fonts** tab.
2. The global **Font** menu, if set to anything other than "from file".
3. A `.def` mapping for the font name (experimental, see below).
4. A loaded font file whose name appears in the AFP font name (for example `Arial.ttf` for a font called `ARIAL`).
5. The alias list: Arial and Helvetica map to Liberation Sans, Times to Liberation Serif, Courier to Liberation Mono, Calibri to Carlito, Cambria to Caladea, Verdana and Tahoma to DejaVu Sans, and so on, when those fonts are loaded.
6. A loaded Liberation font matching the guessed style (sans, serif or mono).
7. The system fonts (Arial, Times New Roman, Courier New).

Font size, bold and italic come from the file's Font Descriptor when it has one, otherwise from the font name, otherwise 10 pt. The **Text size** menu scales everything, and the Fonts tab sets exact sizes per font. Overrides are not saved between sessions yet.

### The font pack

`Fonts/` contains 82 free fonts, one font per file, chosen to match the widths of the fonts AFP documents use most (Arial, Helvetica, Times, Courier, Calibri, Cambria, Verdana, Palatino, Bookman and more). See `Fonts/README.txt` for the full list. Commercial fonts such as the real Arial or Times New Roman are not included; if you own them, put them in `Fonts/local/` (ignored by git). A file named after the AFP font takes priority over the substitutes.

### `.def` font maps (experimental)

A `.def` file is read as plain text, one mapping per line. The name is an AFP character-set or coded-font name:

```text
; AFP font name = family, size, style
C0H200B0 = Arial, 9, bold
X0TIMES  = Times New Roman, 10
```

This is an assumed format, not a documented one. Files that are not plain text are ignored.

## Resource files

AFP documents often rely on separate overlays, page segments and images. Select or drop them together with the document: they are joined into one stream and found by the names stored inside them, whatever their extension. Anything referenced but not found is listed in the footer as a missing resource. Loose image files are matched to include-object names by file name (without the extension).

## Export and print

The **Export** menu offers:

- **Save page as PNG** (150 dpi).
- **Save all pages as PDF**. Each page is a 150 dpi image, so text in the PDF is not selectable.
- **Print page**. Printing can be blocked when the viewer runs inside an embedded or sandboxed frame; save a PDF and print that instead.

## Keyboard shortcuts

| Keys | Action |
| --- | --- |
| Right arrow, Page Down | Next page |
| Left arrow, Page Up | Previous page |
| Home, End | First, last page |
| `+`, `-` | Zoom in, out |

## Repository layout

```text
afp-viewer.html    the whole viewer (HTML, CSS and JavaScript)
Fonts/             free font pack, licenses and its own README
Fonts/local/       your own licensed fonts (git-ignored, create it yourself)
samples/public/    AFP files you may share (optional, not ignored by git)
README.md
.gitignore
```

## Limitations

- **AFP font resources are not read** (raster and outline character sets, code pages, coded fonts), and TrueType fonts embedded in the AFP are not used automatically. Text is drawn with substitute fonts, so character widths and spacing can differ from the original. Matching the original font and size gives the best result.
- **Not drawn:** IOCA bi-level images (G4/MMR and similar), GOCA vector graphics and barcodes appear as dashed outlines.
- **Code page:** EBCDIC text is decoded as code page 037. Other code pages can show wrong characters for some symbols.
- **Not applied:** form definitions, page definitions (line data), medium maps, duplex and n-up, page position offsets, object orientation other than text rotation.
- **Not available:** text search and navigation by index tags.
- Files are parsed in memory, so very large files are limited by browser memory.

## Troubleshooting

- **Text looks crowded or overlapping:** the font size is probably too large. Set the size in the Fonts tab, or use the Text size menu.
- **Characters look garbled:** change the Text menu. Documents with TrueType fonts usually need UTF-16.
- **Stray accented letters:** the document likely uses a code page other than 037.
- **A dashed box with a label:** that object type is not supported yet.
- **Footer lists missing resources:** drop those files on the window together with the document.

## Privacy

No file leaves your machine and the page makes no network requests. AFP files often contain personal data, so `.gitignore` excludes them from the repository by default.

## Development

Everything lives in `afp-viewer.html`, split into commented sections: constants and state, structured-field parser, text renderer, objects and resources, fonts, autodetect, font library, viewer UI, export and print, built-in sample, and event wiring. Edit the file and reload the page; there is no tooling to set up. There are no automated tests yet.

## Roadmap

- Read AFP font resources (character sets, code pages, coded fonts) from the Fonts folder for exact spacing.
- Decode IOCA bi-level (G4) images.
- Draw GOCA vector graphics and common barcodes.
- Remember per-font choices between sessions.
- Text search and index-based navigation.

The bundled fonts keep their own licenses, which are included in `Fonts/licenses/`.
