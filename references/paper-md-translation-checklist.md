# Paper Markdown Translation Checklist

## Translation Style

- Translate with model judgment only; do not call external translation tools.
- Prefer Chinese-first study prose with first-use English terms/acronyms in parentheses.
- Keep domain terminology consistent after first use.
- Preserve author names, journal names, grant numbers, software names, locations, and manufacturer details unless a direct Chinese translation is conventional.
- Preserve numbers, signs, units, Greek letters, equation references, and citation numbering exactly.

## Markdown Preservation

- Keep heading hierarchy equivalent to the source.
- Preserve display math blocks and `\tag{}` labels exactly unless the source has a clear typo.
- Preserve Markdown table column count, row count, alignment markers, and numeric values.
- Translate table headers and row labels, but keep method acronyms such as MRE, DWI, DTI, ADC, SE, EPI, HARDI, or task-specific acronyms when they are standard.
- Preserve image order and figure numbering.
- Use relative image links from the translated Markdown to copied assets.

## Figure Localization Pattern

Before calling imagegen:

1. Inspect the local image with `view_image`.
2. Identify the figure's role and all visible English labels.
3. Build an explicit replacement map.
4. State invariants: preserve geometry, curves, grayscale/color maps, arrows, units, numbers, symbols, panel letters, and data values.
5. After generation, copy the selected image from `$CODEX_HOME/generated_images/...` into the output `assets/zh/` folder.

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
$base = Split-Path -Parent $path
$content = Get-Content -Raw -LiteralPath $path
$links = [regex]::Matches($content,'!\[[^\]]*\]\(([^)]+)\)') | ForEach-Object { $_.Groups[1].Value }
$missing = @()
foreach ($link in $links) {
  $full = Join-Path $base ($link -replace '/','\')
  if (-not (Test-Path -LiteralPath $full)) { $missing += $link }
}
[PSCustomObject]@{ ImageLinks = $links.Count; MissingCount = $missing.Count; MissingLinks = ($missing -join '; ') }
```

Compare structure counts:

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

## Final Report

Report:

- The translated Markdown path.
- The original-figure and localized-figure asset folders.
- Whether imagegen was used.
- The validation results for image links and structure counts.
- Any figure-localization imperfections or manual review notes.
