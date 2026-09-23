# OSCP+ Report Generation Workflow

# OSCP+ Report Generation Workflow

Use this workflow to keep the report synchronized with exam notes, then produce the final Markdown and PDF deliverables.

## Source template

Start from the [OSCP Exam Report Template — whoisflynn v3.2](https://github.com/noraj/OSCP-Exam-Report-Template-Markdown/blob/master/src/OSCP-exam-report-template_whoisflynn_v3.2.md). Prefer the **Raw** file when downloading it.

Template repository: [noraj/OSCP-Exam-Report-Template-Markdown](https://github.com/noraj/OSCP-Exam-Report-Template-Markdown)

## Recommended workflow

1. **Download a local copy of the template.** Keep the original untouched so it remains a clean fallback.
2. **Import the Markdown file into Notion.** Use Notion's import flow and choose the Markdown/text importer. Rename the imported page clearly, for example `OSCP Exam Report — Working Copy`.
3. **Write notes into the imported page throughout the exam.** Record the target IP, hostname, ports and services, commands that worked, vulnerability explanation, exact reproduction steps, screenshots, local.txt/proof.txt values, and timestamps.
4. **Keep evidence beside the relevant finding.** Every finding should be reproducible by another person; do not rely on a statement such as “got shell.”
5. **Export the completed page as Markdown.** Use the page menu → **Export** → **Markdown & CSV**. If Notion downloads a ZIP, extract the Markdown file and its accompanying assets into one working directory.
6. **Validate the exported Markdown locally.** Check that headings, code blocks, images, links, and screenshots survived the export. Replace any broken asset paths before converting.
7. **Generate the PDF with Pandoc** using one of the commands below.
8. **Open the final PDF and verify it** before submission: page breaks, screenshots, code readability, table of contents, finding numbering, and that every machine includes the required proof.

## Local file setup

```bash
mkdir -p oscp-report/assets
cd oscp-report
curl -L "https://raw.githubusercontent.com/noraj/OSCP-Exam-Report-Template-Markdown/master/src/OSCP-exam-report-template_whoisflynn_v3.2.md" \
  -o oscp-report-working.md
```

After the Notion export, place the exported Markdown in this directory and keep image/PDF assets under `assets/` (or update the relative paths in the Markdown).

## Direct Markdown → PDF with Pandoc

The following uses GitHub-Flavored Markdown, preserves raw HTML where possible, creates a table of contents, and uses XeLaTeX for better Unicode/font handling:

```bash
pandoc oscp-report-working.md \
  --from=gfm+raw_html \
  --standalone \
  --toc \
  --number-sections \
  --pdf-engine=xelatex \
  --resource-path=".:assets" \
  -V geometry:margin=1in \
  -o oscp-report.pdf
```

If the template contains LaTeX that should be passed through unchanged, try:

```bash
pandoc oscp-report-working.md \
  --from=gfm+raw_html+raw_tex \
  --standalone --toc --number-sections \
  --pdf-engine=xelatex \
  --resource-path=".:assets" \
  -o oscp-report.pdf
```

## Fallback: Markdown → DOCX → PDF

Use this path when the PDF engine is unavailable or the direct conversion fails.

```bash
pandoc oscp-report-working.md \
  --from=gfm+raw_html \
  --standalone \
  --resource-path=".:assets" \
  -o oscp-report.docx
```

Then convert the DOCX to PDF with LibreOffice:

```bash
mkdir -p pdf-output
libreoffice --headless --convert-to pdf \
  --outdir pdf-output oscp-report.docx
mv pdf-output/oscp-report.pdf ./oscp-report.pdf
```

If a reference DOCX is needed for fonts, margins, headers, or footers, create or edit `reference.docx` and add `--reference-doc=reference.docx` to the DOCX command.

## Troubleshooting checklist

- **`xelatex not found`:** use the DOCX fallback, or install a TeX distribution that provides XeLaTeX.
- **Images missing:** confirm the exported image files exist and that Markdown uses paths relative to `oscp-report-working.md`; use `--resource-path=".:assets"`.
- **Raw HTML renders badly:** remove or simplify unsupported HTML, or convert the affected blocks to standard Markdown before exporting.
- **Characters or symbols break:** use `--pdf-engine=xelatex`; avoid relying on fonts that are not installed.
- **Notion export is a ZIP:** extract it first and preserve the relative directory structure for assets.
- **Code blocks are clipped:** reduce the code size, split long commands, or use the DOCX route and adjust the reference document.

## Final proof checklist

- [ ]  Executive summary and scope are complete
- [ ]  Each machine has IP, hostname, enumeration, foothold, privilege escalation, and reproducible steps
- [ ]  Screenshots show shell context (`hostname` + `whoami`/`id`) with the relevant proof
- [ ]  `local.txt` and `proof.txt` values are recorded with host context
- [ ]  AD attack path and Domain Admin compromise are documented, where applicable
- [ ]  Commands, tool output, screenshots, and timestamps are present
- [ ]  No unsupported claims or unexplained “got shell” steps remain
- [ ]  Final Markdown export was checked
- [ ]  Final PDF opens correctly and was visually reviewed