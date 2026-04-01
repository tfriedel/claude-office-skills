# Office Document Skills for Claude Code

Professional Office document creation and editing workflows for the command line, powered by Claude Code.

## What is this?

This repository packages the same Office document manipulation skills used by [Claude desktop](https://support.claude.com/en/articles/12111783-create-and-edit-files-with-claude) for use with [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) (the CLI version). You get the full power of Claude's document creation capabilities in your terminal, ready to integrate with scripts, CI/CD pipelines, or automated workflows.

update 2026-04-01:
This repo was published before Anthropic made these skills public themselves.
For anyone interested in this I suggest you rather have a look at the official repo as the skills there may be getting updates:

https://github.com/anthropics/skills/tree/main/skills


## Supported Formats

- **PowerPoint (PPTX)** - Create presentations from scratch or templates, with HTML-to-PPTX conversion
- **Word (DOCX)** - Edit documents with tracked changes, OOXML manipulation, redlining workflows
- **Excel (XLSX)** - Build financial models with formulas, formatting, and zero-error validation
- **PDF** - Fill forms, merge documents, extract data, convert to images

## Key Capabilities

### PowerPoint

- **HTML-to-PPTX conversion** - Design slides in HTML/CSS, render to PPTX with full formatting
- **Template-based creation** - Rearrange slides, replace text with JSON, preserve formatting
- **Visual validation** - Generate thumbnail grids to catch text cutoff and layout issues
- **OOXML editing** - Direct XML manipulation for precise control

### Word

- **Tracked changes (redlining)** - Professional document editing with change tracking
- **OOXML manipulation** - Add comments, modify structure, preserve formatting
- **Text extraction** - Export content with tracked changes preserved

### Excel

- **Formula-based models** - Working formulas with zero-error requirement
- **Professional formatting** - Color-coded inputs/formulas, custom number formats
- **Data validation** - Years as text, zeros formatted as "-", proper cell styling

### PDF

- **Form filling** - Populate fillable PDFs programmatically
- **Document merging** - Combine multiple PDFs
- **Format conversion** - PPTX to PDF, PDF to images
- **Data extraction** - Pull information from PDF forms and documents

## Getting Started

### Prerequisites

```bash
# Python dependencies
venv/bin/pip install -r requirements.txt

# Node.js dependencies (for html2pptx)
npm install

# System tools (usually pre-installed)
# - LibreOffice (soffice)
# - Poppler (pdftoppm)
# - Pandoc
```

### Using with Claude Code

Simply tell Claude Code what you want to create:

```
> Create a quarterly sales presentation with 5 slides
> Create a powerpoint presentation based on @input/slide_notes.txt
> Edit this Word document and add tracked changes
> Build an Excel financial model for budget projections
> Fill out this PDF form with data from this JSON
```

Claude Code will:

1. Check if a skill exists for your task
2. Read the appropriate `SKILL.md` workflow
3. Execute the workflow step-by-step
4. Save all outputs to `outputs/<document-name>/`

### Manual Usage

All scripts can also be run directly:

```bash
# Create PowerPoint thumbnail grid
venv/bin/python public/pptx/scripts/thumbnail.py template.pptx outputs/review/thumbnails

# Rearrange slides
venv/bin/python public/pptx/scripts/rearrange.py template.pptx outputs/deck/final.pptx 0,5,5,12,3

# Extract text inventory
venv/bin/python public/pptx/scripts/inventory.py deck.pptx outputs/deck/inventory.json

# Replace text from JSON
venv/bin/python public/pptx/scripts/replace.py input.pptx outputs/deck/replacements.json outputs/deck/output.pptx
```

## Repository Structure

```
public/
├── pptx/           # PowerPoint workflows
│   ├── SKILL.md    # Main workflow documentation
│   ├── html2pptx.md # HTML-to-PPTX guide
│   ├── ooxml.md    # OOXML editing guide
│   └── scripts/    # Python/JS utilities
├── docx/           # Word workflows
├── pdf/            # PDF workflows
└── xlsx/           # Excel workflows

outputs/            # Your generated documents (gitignored)
└── <project-name>/ # One directory per document
```

## How It Works

Each format has a `SKILL.md` file that defines the workflow. Claude Code:

1. **Checks for skills** - Before writing custom code, checks if a skill exists
2. **Reads the skill** - Loads the complete workflow from `SKILL.md`
3. **Follows the workflow** - Executes each step precisely
4. **Validates outputs** - Runs validation scripts (OOXML formats)
5. **Organizes files** - All outputs go to `outputs/<document-name>/`

### Example: Creating a Presentation from Template

```bash
# 1. Extract template text
venv/bin/python -m markitdown template.pptx

# 2. Generate thumbnails
venv/bin/python public/pptx/scripts/thumbnail.py template.pptx outputs/sales-deck/thumbnails

# 3. Rearrange slides
venv/bin/python public/pptx/scripts/rearrange.py template.pptx outputs/sales-deck/working.pptx 0,15,15,23,8

# 4. Extract text inventory
venv/bin/python public/pptx/scripts/inventory.py outputs/sales-deck/working.pptx outputs/sales-deck/inventory.json

# 5. Generate replacement JSON (with formatting)
# Creates outputs/sales-deck/replacements.json

# 6. Apply replacements
venv/bin/python public/pptx/scripts/replace.py outputs/sales-deck/working.pptx outputs/sales-deck/replacements.json outputs/sales-deck/final.pptx
```

Claude Code handles all these steps automatically when you ask it to create a presentation.

## Why Use This?

### Desktop/Web Claude is Great For

- Interactive document creation
- Visual feedback during creation

### Claude Code is Great For

- **Automation** - Generate monthly reports, process batches of documents
- **Custom workflows** - Combine with other tools (databases, APIs, scripts)
- **Server environments** - Run headless without desktop GUI
- **Template iteration** - Rapidly test changes to document templates

## Use Cases

- **Automated reporting** - Generate weekly/monthly presentations from database data
- **Batch processing** - Convert 100 HTML pages to PPTX slides
- Create sales decks based on product data you pulled from a RAG system
- **Document pipelines** - Pull data → populate Excel → generate PDF → email
- **API integration** - Webhook triggers document generation
- learn how to build similar agents for other tasks

## Documentation

- **Getting started**: See `CLAUDE.md` for repository conventions
- **Workflows**: Each `public/*/SKILL.md` defines complete workflows
- **Skills system**: `skills-system.md` explains the skills-check pattern
- **Claude Code docs**: [docs.claude.com/claude-code](https://docs.claude.com/en/docs/claude-code/overview)
- **Desktop version**: [Create and edit files with Claude](https://support.claude.com/en/articles/12111783-create-and-edit-files-with-claude)

## Output Directory Convention

All generated files go to `outputs/<document-name>/`:

```
outputs/
├── quarterly-sales-report/
│   ├── final.pptx
│   ├── thumbnails_grid.png
│   ├── inventory.json
│   └── replacements.json
├── employee-handbook/
│   ├── handbook.docx
│   └── unpacked/
└── budget-2024/
    └── budget.xlsx
```

This keeps your working directory clean and makes automation easier.

## Attribution

Most scripts and workflows in this repository come directly from Claude (Anthropic's AI assistant) and are included here verbatim. If Anthropic wishes for this repository to be taken down, please contact me and I will comply immediately.

## Contributing

This is a skills repository. To add capabilities:

1. Add scripts to `public/<format>/scripts/`
2. Document in the appropriate `SKILL.md`
3. Update `CLAUDE.md` with new commands
4. Ensure validation scripts pass
