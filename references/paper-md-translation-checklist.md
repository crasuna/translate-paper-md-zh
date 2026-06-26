# Paper Markdown Translation Checklist

## Translation Style

- Translate with model judgment only; do not call external translation tools.
- Prefer Chinese-first study prose with first-use English terms/acronyms in parentheses.
- Keep domain terminology consistent after first use.
- Preserve author names, journal names, grant numbers, software names, locations, and manufacturer details.
- Preserve numbers, signs, units, Greek letters, equation references, and citation numbering exactly.

## Markdown Preservation

- Keep heading hierarchy equivalent to the source.
- Preserve display math blocks and `\tag{}` labels exactly.
- Preserve Markdown table column count, row count, alignment markers, and numeric values.
- Translate table headers and row labels, but keep method acronyms that appear in the source, including MRE, DWI, DTI, ADC, SE, EPI, HARDI, or task-specific acronyms.
- Preserve image order and figure numbering.
- Use relative image links from the translated Markdown to copied assets.

## Mechanical Corrections

- Make only non-semantic mechanical corrections: obvious spelling typos in prose, broken Markdown table/link/escape syntax, duplicate punctuation, or OCR/layout residue.
- Do not alter numbers, units, equation meaning, citation numbering, author information, manufacturer details, or scientific claims.
- Report every correction in the final report with the source location and the changed text or syntax.

## Figure Localization Pattern

Before calling imagegen:

1. Inspect the local image with `view_image`.
2. Identify the figure's role and all visible English labels.
3. If the figure has no translatable English labels, copy only the original figure and skip imagegen.
4. Build an explicit replacement map for every translatable English label.
5. State invariants: preserve geometry, curves, grayscale/color maps, arrows, units, numbers, symbols, panel letters, and data values.
6. After generation, copy the selected image from `$CODEX_HOME/generated_images/...` into the output `assets/zh/` folder.
7. Inspect the localized figure before linking it. If labels are unreadable, numbers/units changed, or scientific content was redrawn, discard the result and retry at most once; if it still fails, keep only the original figure and report the residual risk.

English label rule:

- Translate only visible English words or phrases in the figure.
- Do not translate coordinate values, units, acronyms, formula symbols, panel letters, or data labels unless the user provides an explicit replacement map.
- If a label is ambiguous, leave it unchanged and list it in the final report.

## Image Resource Handling

- Copy existing local relative image paths and existing local absolute image paths into `assets/original/`.
- Resolve relative paths from the source Markdown file's directory, not from the current shell directory.
- Do not download URL images by default. Keep the original URL in the translated Markdown and list it under `ExternalResources`.
- Do not create placeholder image files for missing, unreadable, or otherwise uncopyable local resources. Keep the original link and list it under `UncopiedResources`.
- Download or transform external resources only when the user explicitly asks for that behavior.

## Figure Naming

- Copy originals to `assets/original/` using the source file name.
- If two source figures share the same file name, append `_2`, `_3`, and so on before the extension.
- Save accepted localized figures to `assets/zh/` as `<original-stem>_zh.<ext>`.
- If localized names collide, append `_2`, `_3`, and so on before the extension.

Prompt skeleton:

```text
Use case: text-localization
Asset type: Chinese learning version of a scientific paper figure
Input image: the just-shown figure is the edit target.
Primary request: Translate only the English labels into Chinese while preserving the scientific diagram, data, layout, and style.
Replace text exactly as follows: <English -> Chinese map>.
Keep unchanged: <symbols, numbers, units, acronyms, panel labels, curves, arrows, maps>.
Typography: clear black Chinese labels positioned near the original labels.
Constraints: no extra labels, no watermark, no cropping, no changes to scientific data.
```

## QA Commands

Use these PowerShell snippets with real paths substituted.

Check image links:

```powershell
$path = '<translated-md>'
$base = (Resolve-Path -LiteralPath (Split-Path -Parent $path)).Path
$content = Get-Content -Raw -LiteralPath $path
$links = [regex]::Matches($content,'!\[[^\]]*\]\(([^)]+)\)') | ForEach-Object { $_.Groups[1].Value }
$missing = @()
$outside = @()
$external = @()
foreach ($link in $links) {
  $normalized = $link.Trim('"') -replace '/','\'
  if ($normalized -match '^https?://' -or $normalized -match '^[a-zA-Z][a-zA-Z0-9+.-]*://') {
    $external += $link
    continue
  }
  if ([System.IO.Path]::IsPathRooted($normalized)) {
    $outside += $link
    continue
  }
  $candidate = Join-Path $base $normalized
  if (-not (Test-Path -LiteralPath $candidate)) {
    $missing += $link
    continue
  }
  $resolved = (Resolve-Path -LiteralPath $candidate).Path
  $inside = $resolved.Equals($base, [StringComparison]::OrdinalIgnoreCase) -or
    $resolved.StartsWith($base + [IO.Path]::DirectorySeparatorChar, [StringComparison]::OrdinalIgnoreCase)
  if (-not $inside) { $outside += $link }
}
[PSCustomObject]@{
  ImageLinks = $links.Count
  MissingCount = $missing.Count
  OutsideBaseCount = $outside.Count
  ExternalCount = $external.Count
  MissingLinks = ($missing -join '; ')
  OutsideBaseLinks = ($outside -join '; ')
  ExternalLinks = ($external -join '; ')
}
```

Compare structure counts:

Image count expectations: let source image count be `S` and accepted localized figure count be `L`. The translated Markdown image count must be `S + L`.

```powershell
function Counts($label, $path) {
  $c = Get-Content -Raw -LiteralPath $path
  [PSCustomObject]@{
    Label = $label
    Headings = ([regex]::Matches($c,'(?m)^#+ ')).Count
    Images = ([regex]::Matches($c,'!\[')).Count
    FormulaDelims = ([regex]::Matches($c,'(?m)^\$\$')).Count
    Tags = ([regex]::Matches($c,'\\tag\{')).Count
    TableRows = ([regex]::Matches($c,'(?m)^\|')).Count
  }
}
@(Counts 'source' '<source-md>'; Counts 'zh' '<translated-md>') | Format-Table -AutoSize
```

Inspect image dimensions:

```powershell
Add-Type -AssemblyName System.Drawing
Get-ChildItem -LiteralPath '<output-folder>\assets' -Recurse -File -Filter '*.png' |
  ForEach-Object {
    $img = [System.Drawing.Image]::FromFile($_.FullName)
    [PSCustomObject]@{ Name = $_.FullName; Width = $img.Width; Height = $img.Height; Bytes = $_.Length }
    $img.Dispose()
  } | Format-Table -AutoSize
```

## Minimal Forward Test

Use a temporary folder and do not call imagegen for this smoke test.

1. Create a tiny source Markdown with one heading, one paragraph, one Markdown table row, one display math block, and one local image link.
2. Use a local PNG with no visible English text, or omit imagegen entirely if no image tool is available.
3. Run the skill against that Markdown and write output to a separate temporary output folder.
4. Confirm the translated Markdown exists, the original image was copied to `assets/original/`, no localized image was created for the no-English figure, and the image-link check reports `MissingCount = 0`.
5. Delete the temporary folder after reviewing the output.

## Final Report

Report:

- The translated Markdown path.
- The original-figure and localized-figure asset folders.
- Whether imagegen was used.
- The validation results for image links and structure counts.
- `ExternalResources` and `UncopiedResources`.
- Mechanical corrections made, with source location and changed text or syntax.
- Accepted localized figures and unconfirmed localized figures.
- Any figure-localization imperfections or manual review notes.
