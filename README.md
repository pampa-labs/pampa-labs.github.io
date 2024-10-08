# Pampa Index Setup

This guide will help you set up and contribute to the Pampa Labs documentation using MkDocs Material.

## Setup Instructions

1. Clone the repository locally


2. Create a virtual environment and activate it:
   ```
   python -m venv venv
   source venv/bin/activate  
   ```

3. Install MkDocs Material:
   ```
   pip install mkdocs-material
   ```

   For more detailed installation instructions, refer to the [official MkDocs Material documentation](https://squidfunk.github.io/mkdocs-material/getting-started/).

## Adding Content

1. Create a new Markdown file in the `docs` directory:
   ```
   docs/your-new-page.md
   ```

2. Commit your changes and push to the main branch:


The GitHub Actions workflow will automatically build and deploy your changes to the documentation site.

## Local Development

To preview your changes locally before pushing:

1. Run the MkDocs development server:
   ```
   mkdocs serve
   ```

2. Open your browser and go to `http://localhost:8000` to see the live preview of your documentation.

