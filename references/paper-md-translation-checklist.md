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
6. Inspect the localized figure before linking it. If labels are unreadable, numbers/units changed, or scientific content was redrawn, discard the result and retry at most once; if it still fails, keep only the original figure and report the residual risk.

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
foreach ($link in $links) {
  $normalized = $link.Trim('"') -replace '/','\'
  if ($normalized -match '^[a-zA-Z][a-zA-Z0-9+.-]*:' -or [System.IO.Path]::IsPathRooted($normalized)) {
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
  MissingLinks = ($missing -join '; ')
  OutsideBaseLinks = ($outside -join '; ')
}
```

Compare structure counts:

Image count expectations: if the translated Markdown only copies original figures, `Images` should match the source. If it shows both `原图` and `中文标注图`, translated `Images` will usually be twice the source figure count.

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
