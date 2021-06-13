# CMS Migration

1. Markdown conversions

   - Rename from mdtext to md
   - All files must have a title
   - Metadata is a single line
   - Tables must not have spaces in `|---|---|`
   - Fenced code blocks differ
   - Use internal hyperlinks within the site.

   See [changes.txt](changes.txt)

2. Theme Template

   CMS templates were converted into `base.html`

3. Configuration

   See [pelicanconf.py](../pelicanconf.py)

4. Pelican ASF plugin configuration

   [asfgenid.py](../theme/plugins/asfgenid.py)

   - 'unsafe_tags': False # disallow style, script, and iframe tags
   - 'metadata': False    # no metadata replacement in markdown files
   - 'elements': False    # CMS didn't use mdx_elementid featuresz
   - 'headings': True     # Fix up headings w/ permalinks
   - 'headings_re': r'^h[1-5]'
   - 'permalinks': True,
   - 'toc': True          # uses [TOC]
   - 'toc_headers': r"h[1-5]",
   - 'tables': True       # Fix up for markdown table class
