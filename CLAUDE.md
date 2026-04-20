# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a skills repository for Office document manipulation (PPTX, DOCX, XLSX, PDF). Each skill provides workflows, scripts, and documentation for working with specific file formats. The repository follows a pattern established in `skills-system.md` which defines mandatory workflows for document creation and editing.

## Repository Structure

```
public/
├── pptx/           # PowerPoint presentation skills
│   ├── SKILL.md    # Main workflow documentation
│   ├── ooxml.md    # OOXML editing guide
│   ├── html2pptx.md # HTML-to-PPTX conversion guide
│   ├── scripts/    # Python and JS utilities
│   └── ooxml/      # OOXML validation and schemas
├── docx/           # Word document skills
│   ├── SKILL.md    # Main workflow documentation
│   ├── ooxml.md    # OOXML editing guide
│   ├── docx-js.md  # JavaScript library documentation
│   └── scripts/    # Python utilities
├── pdf/            # PDF manipulation skills
│   ├── SKILL.md    # Main workflow documentation
│   ├── REFERENCE.md # Advanced features and examples
│   └── FORMS.md    # PDF form filling guide
└── xlsx/           # Excel spreadsheet skills
    └── SKILL.md    # Main workflow documentation

outputs/            # All skill-generated documents (gitignored)
└── <document-name>/ # One directory per document project
    ├── *.pptx      # Final outputs
    ├── *.docx      # Final outputs
    ├── *.pdf       # Final outputs
    ├── *.xlsx      # Final outputs
    ├── unpacked/   # Unpacked OOXML files
    ├── *.json      # Inventories and replacements
    ├── *.html      # HTML slides
    └── images/     # Generated images
```

## Key Architecture Principles

### Skills-Based System
The repository follows a mandatory skills-check system (defined in `skills-system.md`):
1. **Before writing ANY code**: Check if a skill exists for the task
2. **If YES**: Read the corresponding SKILL.md and follow it exactly
3. **If NO**: Only then proceed with custom code

This prevents reinventing workflows that already exist in the skills documentation.

### Two-Phase Approach for Complex Operations
Most OOXML editing workflows follow this pattern:
1. **Unpack**: Extract the Office file to raw XML using `.venv/bin/python ooxml/scripts/unpack.py`
2. **Edit**: Modify XML files directly
3. **Validate**: Check changes using `.venv/bin/python ooxml/scripts/validate.py --original <file>`
4. **Pack**: Repackage to Office file using `.venv/bin/python ooxml/scripts/pack.py`

### Read-First Policy
All SKILL.md files must be read **completely** before starting work. Never set range limits when reading these files - they contain critical workflow steps and validation requirements.

### Output Directory Convention
**MANDATORY**: All files created or edited using any skill workflow MUST be written to:
```
outputs/<fitting-name-for-document>/
```

Where `<fitting-name-for-document>` is a descriptive, lowercase, hyphenated name for the document being worked on.

**Examples**:
- `outputs/quarterly-sales-report/` - for a sales presentation
- `outputs/employee-handbook/` - for a Word document
- `outputs/budget-2024/` - for an Excel file
- `outputs/project-proposal/` - for a PDF document

**Rules**:
1. Create the outputs directory if it doesn't exist
2. All intermediate files (unpacked XML, JSON inventories, HTML files, images, etc.) go in this directory
3. Final output files (PPTX, DOCX, XLSX, PDF) go in this directory
4. Never write skill-generated files to the repository root or public/ directories
5. Use descriptive names that clearly identify the document's purpose

## Development Setup

### Python Environment
The repository uses a Python virtual environment managed by `uv`.

**CRITICAL**: Always use `uv` for environment and package management. NEVER use `pip`, `pip install`, or manually edit the virtual environment.

```bash
# Install/sync all dependencies from pyproject.toml / uv.lock
uv sync

# Add a new package
uv add <package-name>

# Add a development-only package
uv add --dev <package-name>

# Remove a package
uv remove <package-name>

# Upgrade packages
uv lock --upgrade
uv sync
```

**CRITICAL**: Always use `.venv/bin/python` for all Python commands. NEVER use system Python or assume .venv is activated. All Python commands in this document use explicit .venv paths.

### Node.js Dependencies
JavaScript tools for html2pptx workflow:

```bash
# Install dependencies (includes playwright chromium browser)
npm install
```

### System Tools
Required system dependencies (should be pre-installed):
- LibreOffice: `soffice` (for PDF conversion)
- Poppler: `pdftoppm` (for PDF to image conversion)
- Pandoc: `pandoc` (for document text extraction)

## Common Commands

### PowerPoint (PPTX)

**Text extraction**:
```bash
.venv/bin/python -m markitdown file.pptx
```

**Unpack for XML editing**:
```bash
.venv/bin/python public/pptx/ooxml/scripts/unpack.py input.pptx outputs/<document-name>/unpacked/
```

**Validate after editing**:
```bash
.venv/bin/python public/pptx/ooxml/scripts/validate.py outputs/<document-name>/unpacked/ --original input.pptx
```

**Repack to PPTX**:
```bash
.venv/bin/python public/pptx/ooxml/scripts/pack.py outputs/<document-name>/unpacked/ outputs/<document-name>/final.pptx
```

**Create thumbnail grid for visual analysis**:
```bash
.venv/bin/python public/pptx/scripts/thumbnail.py template.pptx outputs/<document-name>/thumbnails [--cols 4]
```

**Rearrange slides (duplicate, reorder, delete)**:
```bash
.venv/bin/python public/pptx/scripts/rearrange.py template.pptx outputs/<document-name>/rearranged.pptx 0,5,5,12,3
```

**Extract text inventory**:
```bash
.venv/bin/python public/pptx/scripts/inventory.py presentation.pptx outputs/<document-name>/inventory.json
```

**Replace text from JSON**:
```bash
.venv/bin/python public/pptx/scripts/replace.py input.pptx outputs/<document-name>/replacements.json outputs/<document-name>/output.pptx
```

**Convert HTML to PPTX** (requires Node.js):
```bash
node script.js  # Uses html2pptx.js library
```

### Word Documents (DOCX)

**Text extraction with tracked changes**:
```bash
pandoc --track-changes=all file.docx -o outputs/<document-name>/extracted.md
```

**Unpack/Validate/Pack**: Same pattern as PPTX above (use `outputs/<document-name>/` directory)

### PDF

**Merge PDFs**:
```python
from pypdf import PdfWriter, PdfReader
writer = PdfWriter()
for pdf in ["doc1.pdf", "doc2.pdf"]:
    reader = PdfReader(pdf)
    for page in reader.pages:
        writer.add_page(page)
with open("outputs/<document-name>/merged.pdf", "wb") as f:
    writer.write(f)
```

**Convert PPTX to PDF** (for visual analysis):
```bash
soffice --headless --convert-to pdf --outdir outputs/<document-name>/ presentation.pptx
```

**Convert PDF to images**:
```bash
pdftoppm -jpeg -r 150 file.pdf outputs/<document-name>/page
```

### Excel (XLSX)

See `public/xlsx/SKILL.md` for comprehensive formula and formatting standards. Key principles:
- Zero formula errors required
- Color coding: Blue=inputs, Black=formulas, Green=internal links, Red=external links
- Format zeros as "-" in number formatting
- Years as text strings, not numbers

## Critical Workflow Notes

### PPTX Creation from Scratch
1. Create output directory: `mkdir -p outputs/<document-name>/`
2. Read `public/pptx/html2pptx.md` completely (no range limits)
3. Design content-informed color palettes (don't use defaults blindly)
4. Create HTML files for each slide in `outputs/<document-name>/` (720pt × 405pt for 16:9)
5. Use `class="placeholder"` for charts/tables to be added via PptxGenJS
6. Rasterize gradients and icons as PNG using Sharp, save to `outputs/<document-name>/images/`
7. Generate presentation to `outputs/<document-name>/presentation.pptx` using `html2pptx.js` library
8. Create thumbnail grid and validate visually for text cutoff, overlap, positioning issues
9. Iterate until all slides are visually correct

### PPTX Creation from Template
1. Create output directory: `mkdir -p outputs/<document-name>/`
2. Extract text: `.venv/bin/python -m markitdown template.pptx`
3. Create thumbnail grid: `.venv/bin/python public/pptx/scripts/thumbnail.py template.pptx outputs/<document-name>/template`
4. Analyze template and save inventory to `outputs/<document-name>/template-inventory.md` (list ALL slides with 0-based indices)
5. Create outline with template mapping (verify slide indices are within range)
6. Rearrange slides: `.venv/bin/python public/pptx/scripts/rearrange.py template.pptx outputs/<document-name>/working.pptx 0,34,34,50,52`
7. Extract text inventory: `.venv/bin/python public/pptx/scripts/inventory.py outputs/<document-name>/working.pptx outputs/<document-name>/text-inventory.json`
8. Read entire `text-inventory.json` (no range limits)
9. Generate replacement JSON to `outputs/<document-name>/replacements.json` with proper paragraph formatting (bold, bullets, alignment, colors)
10. Apply replacements: `.venv/bin/python public/pptx/scripts/replace.py outputs/<document-name>/working.pptx outputs/<document-name>/replacements.json outputs/<document-name>/final.pptx`

**CRITICAL**: Shapes not listed in replacement JSON are automatically cleared. Only shapes with "paragraphs" field get new content.

### DOCX Editing
- **Someone else's document or formal docs**: Use "Redlining workflow" (tracked changes)
- **Your own document + simple changes**: Use "Basic OOXML editing"
- Always preserve existing formatting and document structure

### File Organization
All skill-based workflows follow the **Output Directory Convention** (see Key Architecture Principles above):
- Working files → `outputs/<document-name>/`
- Final outputs → `outputs/<document-name>/`
- Never commit outputs to git (outputs/ is gitignored)

## Validation is Mandatory

After any OOXML editing (PPTX/DOCX), **always validate immediately**:
```bash
.venv/bin/python public/[format]/ooxml/scripts/validate.py <dir> --original <file>
```

Fix validation errors before proceeding. Never pack a file without validating first.

## Dependencies

### Python Packages (requirements.txt)
- **Office formats**: `python-pptx`, `openpyxl`, `pypdf`
- **XML processing**: `defusedxml`, `lxml`
- **Images**: `Pillow`, `pdf2image`
- **Conversion**: `markitdown`
- **Utilities**: `six`

### Node.js Packages (package.json)
- **Presentation generation**: `pptxgenjs` (v4.0.1)
- **HTML rendering**: `playwright` (includes Chromium)
- **Image processing**: `sharp`
- **Icons**: `react`, `react-dom`, `react-icons`

### System Tools
- **LibreOffice**: `soffice` - PPTX to PDF conversion
- **Poppler**: `pdftoppm` - PDF to image conversion
- **Pandoc**: `pandoc` - Document text extraction with tracked changes

## Code Style

When generating code for document operations:
- Write concise code
- Avoid verbose variable names
- Minimize print statements
- Follow existing patterns in scripts/
