---
id: "2d7f9a3c-6b41-4e8d-9c2a-5f1b3e7a4c90"
name: ocr-to-text
version: "0.1.0"
enabled: true
display-name: OCR to Text
task: Drop an image file to extract text locally with Tesseract and save a .txt output.
icon: scan-text
llm-hint: "Fires on file drop. Runs Tesseract OCR on a local image (png/jpg/jpeg/tiff/bmp/gif/webp) and saves the extracted text as a .txt file in ~/Documents/ocr/. Returns {saved_path}. Local-only, no network."
tags: [ocr, image-to-text, tesseract, on-drop, text-extraction]
trigger:
  type: on-drop
permissions:
  fs:
    - mkdir: ~/Documents/ocr/
    - write: ~/Documents/ocr/
  sidecars: [tesseract]
  input-types: [png, jpg, jpeg, tiff, bmp, gif, webp]
composable: true
outputs:
  - name: saved_path
    type: string
    description: Path to the generated OCR text file
---

# OCR to Text

Drop an image file to extract text with Tesseract and save a `.txt` file in `~/Documents/ocr/`.

```cwidget
1. cwidget fs mkdir ~/Documents/ocr/ --parents
2. $BASE = cwidget text replace "$DROPPED_FILE" --regex "^.*/" --with ""
3. $STEM = cwidget text replace "$BASE" --regex "\.[^.]+$" --with ""
4. $OUT_BASE = cwidget concat ~/Documents/ocr/ "$STEM"
5. cwidget sidecar run tesseract "$DROPPED_FILE" "$OUT_BASE"
6. cwidget ui notify "OCR complete."
7. $OUT = cwidget json set "{}" --path saved_path --value "$OUT_BASE.txt"
```
