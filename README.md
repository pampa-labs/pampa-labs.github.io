# Pampa Labs Technical Blog

This repository contains the source for the Pampa Labs Technical Blog, built using MkDocs Material.

## Setup Instructions

1. Clone the repository:
   ```
   git clone <repository-url>
   cd <repository-name>
   ```

2. Create a virtual environment and activate it:
   ```
   python -m venv venv
   source venv/bin/activate  # On Windows, use `venv\Scripts\activate`
   ```

3. Install dependencies:
   ```
   pip install -r requirements.txt
   ```

## Adding Content

1. Create a new Markdown file in the appropriate subdirectory under `docs/`:
   ```
   docs/category/your-new-page.md
   ```

2. Update the `nav` section in `mkdocs.yml` if adding a new category or top-level page.

3. Add the blog entry to the `nav` section in `mkdocs.yml`. The corresponding place depends on the category of your blog post:
   - For technical writings, add it under the "Technical Writings" section.
   - For new top-level categories, add a new section at the appropriate level.

   Example:
   ```yaml
   nav:
     - Home: index.md
     - Technical Writings:
       - "technical-writings/existing-post.md"
       - "technical-writings/your-new-post.md"  # Add your new post here
   ```

4. Commit your changes and push to the main branch.

The GitHub Actions workflow will automatically build and deploy your changes to the documentation site.

## Local Development

To preview your changes locally before pushing:

1. Run the MkDocs development server:
   ```
   mkdocs serve
   ```

2. Open your browser and go to `http://localhost:8000` to see the live preview of your documentation.

## Deployment

The site is automatically deployed via GitHub Actions when changes are pushed to the main branch. The workflow configuration can be found in `.github/workflows/ci.yml`.
