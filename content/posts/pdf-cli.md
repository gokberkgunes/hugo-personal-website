---
title: "PDF and Document Manipulation via CLI"
date: 2026-03-23T01:25:00-05:00
authors: ["Gokberk Gunes"]
tags: ["PDF", "CLI"]
draft: true
---

## PDF Manipulation (PDFTK)
PDFTK is the essential utility for merging, splitting, and rearranging PDF files.

| Action | Command |
| :--- | :--- |
| **Extract Single Page** | `pdftk input.pdf cat 64 output out.pdf` |
| **Insert PDF in Middle** | `pdftk A=main.pdf B=insert.pdf cat A1-62 B A63-end output merged.pdf` |
| **Merge Multiple Files** | `pdftk file1.pdf file2.pdf cat output combined.pdf` |
| **Rotate All Pages** | `pdftk in.pdf cat 1-endnorth output rotated.pdf` |

## Document Conversion and Reading

### EPUB Management
While conversion tools exist, reading EPUBs directly preserves layout and hyperlinking better than a flattened PDF.

* **Native EPUB Reading:** Use `zathura` with the `mupdf` backend.
    ```bash
    sudo pacman -S zathura-pdf-mupdf
    ```
* **EPUB to PDF Conversion:** If hard-copy conversion is required (note: quality may vary).
    ```bash
    # Install from AUR
    paru -S epub2pdf
    epub2pdf.sh path_to_file.epub
    ```

### Office Documents (DOC/DOCX)
LibreOffice's headless mode is the most reliable way to convert proprietary Microsoft formats to PDF without a GUI.

* **Single File Conversion:**
    ```bash
    libreoffice --headless --convert-to pdf filename.docx
    ```
* **Batch Conversion:**
    ```bash
    libreoffice --headless --convert-to pdf *.docx
    ```
### PDF to DOCX
This is not optimal but it's possible to cut PDF and paste it to `.odf` per other people's
request.
```bash
#!/usr/bin/bash

magick -density 200 "$1" -trim +repage -quality 50 output_%03d.jpg

for i in output_*.jpg; do
	#echo "![Page Image]($i){width=6.5in height=9in}"
	echo "![Page Image]($i){width=6.5in}"
done | pandoc -f markdown-implicit_figures -V geometry:margin=0in -o "$1-converted.odt"
#    | pandoc --dpi 100 -f markdown-implicit_figures -o "$1-converted.docx"


```

## Summary Table

| Source Format | Target Format | Recommended Tool |
| :--- | :--- | :--- |
| .pdf (Edit) | .pdf | `pdftk` |
| .epub | .pdf | `epub2pdf` |
| .doc / .docx | .pdf | `libreoffice --headless` |
| .epub | Direct Read | `zathura-pdf-mupdf` |

> **Tech Note:** If you are running these on a headless server (like a remote ARM node), ensure `dbus-x11` or a virtual frame buffer is available for LibreOffice to initialize its conversion engine.
