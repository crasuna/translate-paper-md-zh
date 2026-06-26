---
name: translate-paper-md-zh
description: Translate English academic paper Markdown files into Chinese study editions. Use when Codex is asked to translate an English paper stored as .md, preserve Markdown structure, formulas, tables, citations, units, and figure order, or produce a Chinese-learning copy with original figures plus localized Chinese figure images using imagegen.
---

# Translate Paper Markdown to Chinese

## Workflow

1. Inspect the source Markdown before editing.
   - Read the file structure, heading levels, image links, tables, math blocks, and citation style.
   - Count headings, image links, math delimiters, `\tag{}` occurrences, and table rows for later comparison.
   - Resolve image paths relative to the source Markdown file.

2. Choose a non-destructive output layout.
   - Use the user's requested output directory when provided.
   - Otherwise create a sibling folder named `<source-stem>_zh`.
   - Create `assets/original/` for copied source figures and `assets/zh/` for localized figure images.
   - Do not overwrite the source Markdown or original image files unless the user explicitly requests it.

3. Translate the Markdown as a Chinese study edition.
   - Write Chinese-first prose; preserve important English terms or acronyms on first use, such as `磁共振弹性成像（MR elastography, MRE）`.
   - Preserve formulas, `\tag{}` labels, citation numbers, units, numeric values, author names, journal metadata, and table geometry.
   - Translate headings, abstract labels, body text, captions, table headings, and explanatory appendix prose.
   - Keep Markdown readable and stable; avoid adding commentary unless the user asked for a study-note or annotated edition.

4. Handle figures.
   - Copy every original figure into `assets/original/` using the source file name; if a name collides, append `_2`, `_3`, and so on before the extension.
   - Inspect each figure for visible, translatable English text labels.
   - For figures with translatable English labels, use the `imagegen` skill's built-in image editing path for `text-localization`.
   - For figures without translatable English labels, do not create a localized figure.
   - For each figure, inspect it first, list exact text replacements, and prompt image editing to change only labels while preserving scientific content.
   - Save final localized images in `assets/zh/` as `<original-stem>_zh.<ext>`; if a name collides, append `_2`, `_3`, and so on before the extension.
   - If a localized figure has unreadable labels, changed numbers/units, or materially redrawn scientific content, discard it and retry at most once; if it still fails, keep only the original figure and report the issue.
   - In the Markdown, show `原图` for every source figure and add `中文标注图` only for accepted localized figures.

5. Validate before finishing.
   - Verify every Markdown image link exists and resolves inside the output folder; flag absolute paths, URLs, or `..` paths that escape the translated edition.
   - Compare source and translated structure counts: headings, math block delimiters, equation tags, and table rows.
   - Preview every localized figure before accepting it and check that labels are readable, numbers/units remain unchanged, and scientific diagrams were not materially redrawn.
   - If a localized figure cannot be previewed, do not report it as accepted; keep the original figure authoritative and report the localized file as unconfirmed risk only if the user explicitly requested unverified localized figures.
   - Report the final Markdown path, figure asset paths, imagegen usage, and any residual risk.

## Reference

Read `references/paper-md-translation-checklist.md` when performing an actual translation or when composing image-localization prompts. It contains the compact QA checklist, reusable PowerShell checks, and the figure-edit prompt pattern.
